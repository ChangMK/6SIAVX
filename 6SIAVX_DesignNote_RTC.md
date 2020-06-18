
# RTC DS3231安裝
- 晶片型號為 __Zynq-7000 xc7z020clg400-1__
- PS端DDR3、型號選 MT41K256M16 RE-15E
- 在vivado裡創建一個小系統，如下圖．在6SIAVX裡DS3231是透過I2C控制
![vivado Block][1]
- PS端分配圖如下
  - 勾選I2C0、 UART1、 ENET0 、QUAD SPI 
    - I2C0 透過EMIO到PL腳位，腳位設定**scl**  __\<u18>__ 、**sda** __\<v15>__
![分配圖][2]
  -  ENET0 使用內定MIO腳位外，另外還會使用到MDIO，所以MDIO也需要勾選
![MDIO][3]
- constrain.xdc file如下
``` 
    set_property IOSTANDARD LVCMOS33 [get_ports IIC_0_scl_io]
    set_property IOSTANDARD LVCMOS33 [get_ports IIC_0_sda_io]
    set_property PACKAGE_PIN U18 [get_ports IIC_0_scl_io]
    set_property PACKAGE_PIN V15 [get_ports IIC_0_sda_io]
    set_property PULLUP true [get_ports IIC_0_scl_io]
    set_property PULLUP true [get_ports IIC_0_sda_io]
```
- 設定完畢後按validate design
    ![validate][4]
- 在如下圖中的`Sources`點選 __Create HDL Wrapper__ 產生top file
    ![Create HDL Wrapper][5]
- 接下來產生.bit檔．然後選擇 File->export->Export Hardware 因為有使用到PL端的腳位(EMIO)，記得要把 __Include bitstream__ 一同勾選，這樣XSA檔才會包含bitstream檔． 
- 使用petalinux導入剛產生的XSA檔，創建一個叫minisystem的project
- 修改deveice-tree其名稱為`system-usr.dtsi`，它位於<plnx-proj-root>/ project-spec/meta-user/recipes-bsp/device-tree/files，修改成如下所示
    ```
    /include/ "system-conf.dtsi"
    / {
    };
    &i2c0 {
         clock-frequency = <400000>;
         status = "okay";
         /* DS1337 RTC module */
         rtc0@68 {
                   compatible = "dallas,ds1337";
                   reg = <0x68>;
         };
    };
    ```
    - 因為我們的rtc要透過i2c0控制，所以要掛在它底下，同時建立一個rtc0的設備名稱，它的i2c位址是0x68，使用的驅動名稱要設為`dallas,ds1337`，ds1337 ds1307 ds3231都是使用同一個驅動，所以可以互相更改

[1]: ./png/vivado_IIC_0.png
[2]: ./png/vivado_PS.png
[3]: ./png/vivado_IIC_ETH_MIO.png
[4]: ./png/vivado_validate.png
[5]: ./png/vivado_create_HDL_weapper.png







