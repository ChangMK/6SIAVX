
# vivado 匯入自訂IP
## 方法

![][1]
![][2]
設定放IP的目錄，vivado會自動往目錄下找所有的IP
![][3]

# EMMC使用
- 在vivado中設定使用EMMC（在picozed裡EMMC是SD1)
![][4]
- 在設置petalinux-config 主SD為1(SD1)後，uboot啓動指令沒有做出相對應的修改，這算是官方軟體的bug，所以要手動修`<plnx-proj-root>/project-spec/meta-plnx-generated/recipes-bsp/u-boot/configs/platform-auto.h` ，將sdbootdev修改等於1，如下圖．[詳細參考][6]
  
  ![][5]

  

[1]: ./png/vivado_IP_import1.png
[2]: ./png/vivado_IP_import2.png
[3]: ./png/vivado_IP_import3.png
[4]: ./png/vivado_EMMC1.png
[5]: ./png/vivado_EMMC2.png
[6]: http://news.migage.com/articles/ZYNQ7000+%235++%E4%BB%8Evivado%E5%B7%A5%E7%A8%8B%E5%BC%80%E5%A7%8B%EF%BC%8C%E4%BB%8Eemmc%E5%90%AF%E5%8A%A8Linux_2425673_csdn.html