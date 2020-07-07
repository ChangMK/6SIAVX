- [i2ctools工具安裝](#i2ctools工具安裝)
- [NFS使用 (ubuntu 16.04)](#nfs使用-ubuntu-1604)
- [在rootfs下新增自訂目錄](#在rootfs下新增自訂目錄)
- [LCD設定方式](#lcd設定方式)

# i2ctools工具安裝
```
    petalinux-config -c rootfs
```
>&nbsp; Filesystem Packages --->
>>&nbsp; Base --->
>>>&nbsp; i2c-tools
>>>>&nbsp; <*> i2c-tools

# NFS使用 (ubuntu 16.04)
- 在伺服端建立NFS server
  1. 確定firewall 關閉
        ```
        sudo ufw disable
        ```

  2. 安裝NFS
        ```
        sudo apt-get install nfs-kernel-server nfs-common
        ```
  3. 設定分享目錄 **`按照自己需求設定目錄`**
        ```
        mkdir nfs_share ; chmod -R 777 nfs_share
        ```

  4. 到 /etc/exports 設定分享目錄名稱與權限, 再最後一行加上下行,*表示全部客戶端都可以連進來
        ```
        /home/mk/nfs_share *(rw,sync,no_root_squash)
        ```
  5. 啟動NFS 
        ```
        sudo /etc/init.d/nfs-kernel-server restart
        ```

  6. 安裝rpcbind 
        ```
        sudo apt-get install rpcbind
        ```
  7. 開啓firewall (如果設定好之後，NFS無法運做就再關掉它)
        ```
        sudo ufw enable
        sudo ufw allow 69/udp
        sudo ufw allow 111
        sudo ufw allow 1011
        sudo ufw allow 2049
        ```
        - 從/etc/services可以看出各port作用
           - 69/udp:tftp
           - 111: portmap
           - 1011: mountd
           - 2049: nfs
        - sudo ufw status numbered 可看加入哪些規則

- 設定u-boot掛載NFS
    ```
    petalinux-config
    ```
    >&nbsp; Image Packaging Configuration --->
    >>&nbsp; (X) NFS
    >>>&nbsp; Root filesystem type (NFS) --->
    >>>>&nbsp; Location of NFS root directory
    >>>>&nbsp; NFS Server IP address 
    
![NFS][1]

- 從設定的tftp目錄裡解壓rootfs.tar.gz到設定的NFS目錄裡    
- 重新燒錄u-boot至flash或SD卡

# 在rootfs下新增自訂目錄
- 在應用程式的配方.bb檔裡新增一行
  ```
    FILES_${PN} += "自訂目錄的路徑"
  ```  
    - 例如
    ```
    GMI="/opt/GMI"
    FILES_${PN} += "/opt/GMI"
    do_install() {
	     install -d ${D}/opt/GMI
	     install -m 0755 ${S}/ntpdate ${D}/opt/GMI
	     install -m 0755 ${S}/ntpd ${D}/opt/GMI
         install -m 0755 ${S}/devmem2 ${D}/opt/GMI
	     install -d ${D}/etc
         install -m 0755 ${S}/ntp.conf ${D}/etc
         install -m 0755 ${S}/resolv.conf.static ${D}/etc
    }
    ```
    ![addDirectory][2]

# LCD設定方式
- I2C直接操作
  - `0x43c0007c 0x00030000` or `slv_reg31[17:16]=2'b11` 表示開背燈 
  - `0x43c0007c 0x00020000` or `slv_reg31[17:16]=2'b10` 表示關背燈 
  - `0x43c0007c 0x00000000` or `slv_reg31[17:16]=2'b01` 表示發出reset及開背燈 
  - `0x43c0007c 0x00000000` or `slv_reg31[17:16]=2'b00` 表示發出reset及關背燈 
  - `slv_reg31[27]=1` 表示 Init work
  - `slv_reg31[26]=1` 表示 LCM上指定位置不閃爍
  - `slv_reg31[24]=1` LCM顯示更新
  - `0x43c0007c 0x10000000` or `slv_reg31[28]=1` 表示要直接透過I2C操作LCD
  - `0x43c00040 0x007C000F` or `slv_reg16` 設定好要透過I2C操作後,透過這個位址或暫存器可以寫入操作命令及資料
    - 7C: LCD的I2C位址
    -命令模式: `slv_reg16[15:8]=8'h00`
    -資料模式: `slv_reg16[15:8]=8'h40`
        -Exapme: 寫入'A' 0x43c00040 0x007C4041
    - MSByte
      - 0x2:游標歸位 
      - 0xF:顯示游標及閃爍目前位置
      - 0xE:顯示游標
      - 0xD:目前位置閃爍
      - 0x80:跳到第一行，向右移一格就加1
      - 0xC0:跳到第二行，向右移一格就加1
      - 0xC:顯示螢幕
      - 0x8:關閉螢幕




[1]: ./png/vivado_NFS_client_setting.png
[2]: ./png/vivado_add_new_directory.png


