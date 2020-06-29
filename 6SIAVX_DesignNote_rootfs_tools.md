[i2ctools](#i2ctools工具安裝)
[NFS](#nfs使用)

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
  7. 開啓firewall (如果設定好之後還NFS無法運做就再關掉它)
        ```
        sudo ufw enable
        ```

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







[1]: ./png/vivado_NFS_client_setting.png


