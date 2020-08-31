# Tratamiento-de-audio-digital-con-interfaz-AXI-Stream-e-implementaci-n-en-FPGA
Codigo en VHDL para la implementación de un modificador de audio, con tres etapas, control de volumen, filtrado digital y efecto de audio Delay con entrada y salida por PMOD I2S2
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use ieee.std_logic_unsigned.all;
 
entity axis_i2s2_vhdl is
 Generic ( constant EOF_COUNT :std_logic_vector(8 downto 0) := "111000111" ); 
    Port ( axis_clk    : in std_logic;
           axis_resetn : in std_logic;
          --AXIS SLAVE INTERFACE
           tx_axis_s_data  : in  std_logic_vector (31 downto 0);
           tx_axis_s_valid : in  std_logic;
           tx_axis_s_ready : out std_logic;
           tx_axis_s_last  : in  std_logic;
          --AXIS MASTER INTERFACE
           rx_axis_m_data  : out std_logic_vector (31 downto 0);
           rx_axis_m_valid : out std_logic;
           rx_axis_m_ready : in  std_logic;
           rx_axis_m_last  : out std_logic;
          --I2S TRANSMITTER INTERFACE  
           tx_mclk  : out std_logic;
           tx_lrck  : out std_logic;
           tx_sclk  : out std_logic;
           tx_sdout : out std_logic;
          --I2S RECEIVER INTERFACE   
           rx_mclk : out std_logic;
           rx_lrck : out std_logic;
           rx_sclk : out std_logic;
           rx_sdin : in  std_logic :='0' );
end axis_i2s2_vhdl;
 
architecture Behavioral of axis_i2s2_vhdl is

signal count : std_logic_vector(8 downto 0) := (others => '0');                            
                                              
signal lrck : std_logic; 
signal sclk : std_logic;  
signal mclk : std_logic;
  
signal tx_data_l : std_logic_vector(31 downto 0) := (others => '0');
signal tx_data_r : std_logic_vector(31 downto 0) := (others => '0');
  
signal tx_data_l_shift : std_logic_vector(23 downto 0) :=(others => '0');
signal tx_data_r_shift : std_logic_vector(23 downto 0) :=(others => '0');
  
signal rx_data_l_shift :std_logic_vector(23 downto 0) :=(others => '0');
signal rx_data_r_shift :std_logic_vector(23 downto 0) :=(others => '0');
  
signal rx_data_l :std_logic_vector(31 downto 0) := (others => '0');
signal rx_data_r :std_logic_vector(31 downto 0) := (others => '0');  

signal tx_axis_s_ready_interna : std_logic := '0';
signal rx_axis_m_valid_interna : std_logic := '0';
signal rx_axis_m_last_interna  : std_logic := '0'; 

begin
 
lrck <= count(8);  
sclk <= count(2);   
mclk <= axis_clk;

  tx_lrck <= lrck;  
  tx_sclk <= sclk;
  tx_mclk <= mclk;
  rx_lrck <= lrck;
  rx_sclk <= sclk;
  rx_mclk <= mclk;
   
 tx_axis_s_ready <= tx_axis_s_ready_interna;      
 rx_axis_m_valid <= rx_axis_m_valid_interna; 
 rx_axis_m_last  <= rx_axis_m_last_interna; 
 
process(axis_clk)
begin
    if rising_edge(axis_clk) then  
          count <= count + 1;
    end if;
end process;
  
process(axis_clk)
begin                                       
 if( rising_edge(axis_clk)) then
    if( axis_resetn = '0') then
          tx_axis_s_ready_interna <= '0';  
    elsif (tx_axis_s_ready_interna = '1') and (tx_axis_s_valid = '1')
               and (tx_axis_s_last = '1') then 
          tx_axis_s_ready_interna <= '0';
    elsif (count = "000000000") then  
          tx_axis_s_ready_interna <= '0';
    elsif (count = EOF_COUNT) then 
          tx_axis_s_ready_interna <= '1';   
     end if;
   end if;
end process;
 
process(axis_clk)    
begin
   if( rising_edge(axis_clk)) then
       if (axis_resetn = '0') then    
           tx_data_r <= (others => '0');
           tx_data_l <= (others => '0');
       elsif (tx_axis_s_valid = '1' and tx_axis_s_ready_interna = '1')then 
       if (tx_axis_s_last = '1') then                            
           tx_data_r <= tx_axis_s_data;                           
       else         
          tx_data_l <= tx_axis_s_data;
        end if;     
       end if;
     end if;
  end process;
    
process(axis_clk)
begin
  if (rising_edge(axis_clk)) then
      if (count = "000000111") then
         tx_data_l_shift <= tx_data_l(23 downto 0);
         tx_data_r_shift <= tx_data_r(23 downto 0);
      elsif (count(2 downto 0) = "111") and (count(7 downto 3) >= "00001")
         and (count(7 downto 3) <= "11000") then 
         if(count(8) = '1') then
            tx_data_r_shift <= tx_data_r_shift(22 downto 0) & '0'; 
         else
            tx_data_l_shift <= tx_data_l_shift(22 downto 0) & '0';
         end if;
       end if;
   end if;
end process;
      
process(count,tx_data_r_shift,tx_data_l_shift)
begin
  if(count(7 downto 3)<= "00001") and (count(7 downto 3)>= "11000")then
            if(count(8) = '1') then
                tx_sdout <= tx_data_r_shift(23); 
            else
                tx_sdout <= tx_data_l_shift(23);
            end if;
        else
            tx_sdout <= '0';
        end if;
    end process;  
       
  process(axis_clk)
  begin
     if (rising_edge(axis_clk)) then
       if(count (2 downto 0) = "000") and (count(7 downto 3) >= "00001")
     and (count(7 downto 3) <= "11000") then
         if (lrck = '1') then
             rx_data_r_shift <= rx_data_r_shift(22 downto 0) & rx_sdin;
         else
             rx_data_l_shift <= rx_data_l_shift(22 downto 0) & rx_sdin;
         end if;
        end if;
       end if;
    end process;
    
process(axis_clk)
begin
  if (rising_edge(axis_clk)) then
      if(axis_resetn = '0') then  
         rx_data_l <= (others => '0');
         rx_data_r <= (others => '0');
      elsif (count=EOF_COUNT) and (rx_axis_m_valid_interna='0')then
         rx_data_l <= ("00000000" & rx_data_l_shift);                  
         rx_data_r <= ("00000000" & rx_data_r_shift);
        end if;
        end if;
    end process;
      
  rx_axis_m_data <=  rx_data_l when (rx_axis_m_last_interna = '0')     
                                  else  rx_data_r;  
  process(axis_clk)
  begin                                                  
    if (rising_edge(axis_clk)) then
      if(axis_resetn = '0') then
          rx_axis_m_valid_interna <= '0';
       elsif (count=EOF_COUNT) and (rx_axis_m_valid_interna = '0')then 
            rx_axis_m_valid_interna <= '1';
       elsif (rx_axis_m_valid_interna = '1') and (rx_axis_m_ready ='1')
         and (rx_axis_m_last_interna = '1') then  
                      rx_axis_m_valid_interna <= '0';
            end if;
        end if;
  end process;
    
process(axis_clk)
begin                                                    
  if (rising_edge(axis_clk)) then
     if(axis_resetn = '0') then
         rx_axis_m_last_interna <= '0';  
     elsif(count = EOF_COUNT) and (rx_axis_m_valid_interna = '0') then
         rx_axis_m_last_interna <= '0';
     elsif(rx_axis_m_valid_interna='1') and (rx_axis_m_ready = '1')then
           rx_axis_m_last_interna <= not(rx_axis_m_last_internal);
     end if;
     end if;
    end process;
  
end Behavioral;
