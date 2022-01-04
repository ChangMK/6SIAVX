
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
    - 在\<plnx-proj-root> 輸入
        ```bash
            petalinux-config -c kernel 
        ```
        打開I2C及RTC的驅動
            - 這裡使用的kernel版本是 __4.19.0__ \
        > Device Drivers --->  
        >> &nbsp;I2C  support --->
        >>> &nbsp; I2C Hardware Bus support --->
        >>>> &nbsp; <*>Cadence I2C Controller 

        > Device Drivers --->  
        >> &nbsp;Real Time Clock --->
        >>>&nbsp; <*>Dallas/Maxim DS1307/37/38/39/40/41, ST M41T00, EPAON RX-8025, ISL12057
    
    - 編譯完成後，在開發板可看到下圖
        
        ![RTC][6] 

      使用i2ctools(安裝請看 [這裡](6SIAVX_DesignNote_rootfs_tools.md))可以看到下圖

        ![i2ctools][7]

        68的位置有一個UU，或者使用下面指令讀出rtc的值
        ```
            i2cdump -f -y 0 0x68
        ```
        - 0: i2c bus 0
        - 0x68: rtc的i2c地址(68是出廠預設值)     
  - 預設的系統沒有/etc/localtime這個檔案，若無此檔則系統會以UTC+0的時間來顯示，此檔是根據/usr/share/zoneinfo裡的`區域`裡`城市`做依據，所以只須將此檔複製一份到/etc裡，並改名為 __localtime__
    1. 編輯好一份script file 命名為timezone_change.sh
        ```bash
        #!/bin/bash

        AREA="Asia"
        CITY="Taipei"
        if [ -z "$1" ] || [ -z "$2" ]; then 
        	cp /usr/share/zoneinfo/$AREA/$CITY /etc
            mv /etc/$CITY /etc/localtime
        else
            cp /usr/share/zoneinfo/$1/$2 /etc   
            mv /etc/$2 /etc/localtime
        fi
        ``` 
    2. 利用petalinux建立一個新應用程式，參考`UG1144 Customizing the Rootfs`
        ```bash
        petalinux-create -t apps --name rc.d --template install --enable
        ```    
        使用用途為專門放置開機自動執行的程式
    3. 修改`rc.d.bb`檔，此檔放在\<plnx-proj-root>/project-spec/meta-user/recipes-apps/rc.d目錄裡
         ``` 
         SUMMARY = "Simple rc.d application"
         SECTION = "PETALINUX/apps"
         LICENSE = "MIT"
         LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
         
        inherit update-rc.d

        SRC_URI = "file://timezone_change.sh \
	       "
        RDEPENDS_${PN} += "bash"

        S = "${WORKDIR}"
        INITSCRIPT_PARAMS = "defaults 10"
        INITSCRIPT_NAME = "timezone_change.sh"

        do_install() {
	        install -d ${D}/etc/init.d
	        install -m 0755 ${S}/timezone_change.sh ${D}/etc/init.d
    }
        ```
        - INITSCRIPT_PARAMS = "start 01 S . "  : 重新編譯後燒錄啓動，我們就可以在/etc/rcS.d會有一個S01test.sh的鏈接檔案，指向/etc/init.d/test.sh
        - INITSCRIPT_PARAMS = "defaults 10"  : 在/etc/rc2.d /etc/rc3.d /etc/rc4.d /etc/rc5.d會有一個S10test.sh的鏈接檔案，指向/etc/init.d/test.sh
        - INITSCRIPT_PARAMS = "start 02 2 3 4 5 . stop 01 0 1 6 ." : 相當於使用defauits

[1]: ./png/vivado_IIC_0.png
[2]: ./png/vivado_PS.png
[3]: ./png/vivado_IIC_ETH_MIO.png
[4]: ./png/vivado_validate.png
[5]: ./png/vivado_create_HDL_weapper.png
[6]: ./png/vivado_RTC_installed.png
[7]: ./png/vivado_RTC_installed_i2ctool.png







