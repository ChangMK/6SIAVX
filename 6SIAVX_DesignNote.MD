# 網路校時工具 ntp安裝

## 準備工具 ##
  - ntp [source code](http://www.ntp.org/downloads.html) 這裡我是使用ntp-4.2.8p14.tar.gz版
  - cross compiler [下載網頁](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads) 這裡版本是使用 [gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz?revision=fed31ee5-2ed7-40c8-9e0e-474299a3c4ac&la=en&hash=76DAF56606E7CB66CC5B5B33D8FB90D9F24C9D20)
  - openssl [source code](https://github.com/openssl/openssl.git)
  - DNS設定

## Cross Compiler 安裝
-  解壓至自己指定之目錄
-  將以下三行加入到home目錄底下之 .bashrc最後面  
   >export ARCH=arm 

    >export PATH=/home/mk/DD1/work/xilinx/compiler/bin/:$PATH 

    >export CROSS_COMPILE=arm-none-linux-gnueabihf-
     ``` bash
         echo ARCH=arm >> ~/.bashrc
         echo PATH=/home/mk/DD1/work/xilinx/compiler/bin/:$PATH 
         echo CROSS_COMPILE=arm-none-linux-gnueabihf-
     ```
- 重開終端視窗
  
## openssl 安裝
- 先建立一個openssl_d的目錄
- 進入目錄後輸入下列指令將source code 複製下來
    ```  bash
        git clone https://github.com/openssl/openssl.git 
    ```
-  設定編譯參數
    ```bash
    setarch i386 ./config no-asm no-shared --prefix=/home/mk/DD1/work/playground/openssl --cross-compile-prefix=arm-none-linux-gnueabihf-
    ```
    - setarch i386: 指編譯成32位元，若不指定則會報錯
    - no-shared: 指靜態編譯，只生成靜態庫
    - \--prefix: 指定安裝目錄(我是預先創建好目錄，不知是否能不用預先創建)
    - cross-pile-prefix: 指定交叉編譯器的字首
- 輸入 `make` 產生Makefile檔
- 輸入 `make install_sw` 進行編譯，不會產生文件(?)

## ntp 安裝
- 先建立一個ntp的目錄
- 解壓至自己指定之目錄，進入目錄
-  設定編譯參數
   ```bash
    ./configure --with-yielding-select=yes --host=arm-linux CC=arm-none-linux-gnueabihf-gcc --prefix=/home/mk/DD1/work/playground/ntp/ --with-openssl-incdir=/home/mk/DD1/work/playground/openssl/include
   ```
    - \--prefix: 為安裝目錄
    - \--with-openssl-incdir: openssl靜態庫位址
    - 輸入`make`產生Makefile檔
    - 輸入`make install`進行編譯
    - 編譯完後會產成四個資料夾，我們所需的ntp工具在/ntp/bin目錄裡
    - 可以選擇ntpd或ntpdate來使用，ntpd為建議工具

## ntp 工具安裝進rootfs
- 利用petalinux建立一個新應用程式，參考`UG1144 Customizing the Rootfs`
    ```bash
        petalinux-create -t apps --name ntp --template install --enable
    ```
    - \--template install: 表示使用自行編譯的程式
    - \--template c: 表示使用C語言
  
  1. 將ntp工具放進\<plnx-proj-root>/ project-spec/meta-user/recipes-apps/ntp/files目錄裡，這裡我只使用ntpd和ntpdate
  2. ntpd需要額外一個ntp.conf檔內容是放ntp server的位置，將此檔編輯好之後放進上述目錄
      > server tock.stdtime.gov.tw

      > server watch.stdtime.gov.tw

      > server time.stdtime.gov.tw

      > server clock.stdtime.gov.tw

      > server tick.stdtime.gov.tw
   3. 編輯`ntp.bb`檔，它放在\<plnx-proj-root>/ project-spec/meta-user/recipes-apps/ntp目錄底下
    ```bash
    SUMMARY = "Simple ntp application"
    SECTION = "PETALINUX/apps"
    LICENSE = "MIT"
    LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

    SRC_URI = "file://ntpdate \
           file://ntp.conf \
           file://ntpd    \		
	"
    S = "${WORKDIR}"

    do_install() {
	         install -d ${D}/${bindir}
	         install -m 0755 ${S}/ntpdate ${D}/${bindir}
	         install -m 0755 ${S}/ntpd ${D}/${bindir}
             install -d ${D}/etc
             install -m 0755 ${S}/ntp.conf ${D}/etc
    }
    ```
  - __若要檔案安裝於不同的目錄，參數d與m必須為一組使用__    
  - ${bindir}: 預設為 /usr/bin
  - ${D}: 指準備安裝到映像檔的位置(DEPLOY_DIR_IMAGE)，這裡指的是rootfs
  -  \-d 指定要安裝的目錄名稱，若沒有則創建．相當於mkdir -p
  -  \-m --mode=模式:自行設定權限模式
  -  其餘參數可參考 [linix install命令](https://man.linuxde.net/install) 
   
## DNS與DHCP設定
- 編輯`interfaces`檔，它放在\<plnx-proj-root>/project-spec/meta-plnx-generated/recipes-core/init-ifupdown/files底下
    ```bash
        auto lo
        iface lo inet loopback
        auto eth0
        
        # 動態IP設定
        # iface eth0 inet dhcp

        # 靜態IP設定
        iface eth0 inet static
	    
        address 10.0.1.110
	    netmask 255.0.0.0
	    gateway 10.0.1.101

        # 設定DNS伺服器
        dns-nameserver 8.8.8.8
        # 多組DNS伺服器
        #  dns-nameserver 8.8.8.8 8.8.4.4
    ```





