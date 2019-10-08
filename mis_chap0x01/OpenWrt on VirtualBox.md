# 实验一 VirtualBox的配置与使用

## OpenWrt on VirtualBox

### 实验步骤

+ ```
  # 下载镜像文件
  wget https://downloads.openwrt.org/snapshots/targets/x86/64/openwrt-x86-64-combined-squashfs.img.gz
  # 解压缩
  gunzip openwrt-x86-64-combined-squashfs.img.gz
  # img 格式转换为 Virtualbox 虚拟硬盘格式 vdi
  VBoxManage convertfromraw --format VDI openwrt-x86-64-combined-squashfs.img openwrt-x86-64-combined-squashfs.vdi
  # 新建虚拟机选择「类型」 Linux / 「版本」Linux 2.6 / 3.x / 4.x (64-bit)，填写有意义的虚拟机「名称」
  # 内存设置为 256 MB
  # 使用已有的虚拟硬盘文件 - 「注册」新虚拟硬盘文件选择刚才转换生成的 .vdi 文件
  ```

+ openwrt on virtualbox基本安装

  + 下载镜像文件，在下载镜像时，包含 `squashfs` 关键词的镜像文件区别于包含 `ext4` 关键词的镜像文件之处在于：`squashfs` 镜像包含一个只读文件系统可以用于「恢复出厂设置」。

    <img src="image\镜像下载.png" style="zoom:75%;" />

  + 将镜像文件使用gunzip进行解压缩（此处引入一个工具:

    [chocolatey](https://chocolatey.org/)，--一个很好用的包管理器）

    
    
    <img src="image\choco.png"  />
    
  + 用virtual box自带工具VBoxManage将镜像文件转为vdi,需要注意的是要把VBoxManage加到环境变量中才可以直接在命令行中使用
    
  + 新建虚拟机选择「类型」 Linux / 「版本」Linux 2.6 / 3.x / 4.x (64-bit)，填写有意义的虚拟机「名称」
  
  + 内存设置为 256 MB
    
  + 使用已有的虚拟硬盘文件 - 「注册」新虚拟硬盘文件选择刚才转换生成的 .vdi 文件
    
  + 至此，OpenWrt基本已安装好
    
    <img src="image\openwrt.png"  />
    
  + 接下来进行网卡的设置
  
    - 第一块网卡设置为：Intel PRO/1000 MT 桌面（仅主机(Host-Only)网络）
    - 第二块网卡设置为：Intel PRO/1000 MT 桌面（网络地址转换(NAT)）
  
    <img src="image\两块网卡.png"  />
  
    + 第一块网卡设置如下:
  
      
  
      <img src="image\网卡设置1.png"  />
  
      <img src="image\网卡设置2.png"  />
  
    + 至此，虚拟机的基本配置已完成
    
   + 接下来，启动虚拟机，大约数秒之后（根据宿主机性能不同可能会有差异）黑色命令行界面不再滚动更新新消息时，按下「ENTER」键即可进入 OpenWrt 的终端控制台。
  
   + 检查虚拟机网络的连通性，此处ping通了百度的ip地址但是无法ping通url，接下来进行配置文件的修改后成功
  
     ```
     # 删除 /etc/resolv.conf
     vi /etc/resolv.conf
     # 加入以下
     
     nameserver 114.114.114.114
     nameserver 114.114.114.115
     nameserver 8.8.8.8
     nameserver 8.8.4.4
     
     # reboot
     ```
  
     <img src="image\ping通.png"  />
  
     <img src="image\ping百度.png"  />
  
  + 需要进行配置，此处一定要注意：openwrt的lan端口通常被设为管理端口，对应**内网**，只支持**静态配置ip**，我们暂且修改为192.168.56.10。
  
    ```
    vi /etc/config/network
    ```
  
    <img src="image\修改ip.png"  />
  
  + 配置完成以后重启系统使网络生效
  
    ```
    /etc/init.d/network restart
    ```
  
    + 通过 `OpenWrt` 的软件包管理器 `opkg` 进行联网安装软件
  
      ```
      # 更新 opkg 本地缓存
      opkg update
      
      # 检索指定软件包
      opkg find luci
      # luci - git-19.223.33685-f929298-1
      
      # 查看 luci 依赖的软件包有哪些 
      opkg depends luci
      # luci depends on:
      #     libc
      #     uhttpd
      #     uhttpd-mod-ubus
      #     luci-mod-admin-full
      #     luci-theme-bootstrap
      #     luci-app-firewall
      #     luci-proto-ppp
      #     libiwinfo-lua
      #     luci-proto-ipv6
      
      # 查看系统中已安装软件包
      opkg list-installed
      
      # 安装 luci
      opkg install luci
      
      # 查看 luci-mod-admin-full 在系统上释放的文件有哪些
      opkg files luci-mod-admin-full
      # Package luci-mod-admin-full (git-16.018.33482-3201903-1) is installed on root and has the following files:
      # /usr/lib/lua/luci/view/admin_network/wifi_status.htm
      # /usr/lib/lua/luci/view/admin_system/packages.htm
      # /usr/lib/lua/luci/model/cbi/admin_status/processes.lua
      # /www/luci-static/resources/wireless.svg
      # /usr/lib/lua/luci/model/cbi/admin_system/system.
      # ...
      # /usr/lib/lua/luci/view/admin_network/iface_status.htm
      # /usr/lib/lua/luci/view/admin_uci/revert.htm
      # /usr/lib/lua/luci/model/cbi/admin_network/proto_ahcp.lua
      # /usr/lib/lua/luci/view/admin_uci/changelog.htm
      ```
  
      <img src="image\opkg.png"  />
  
  + 以下是安装好 `luci` 后通过浏览器访问管理 `OpenWrt` 的效果截图。注意：首次不需要密码直接可登录。
  
    <img src="image\luci.png"  />
  
  + 设置密码
  
    <img src="image\设置密码.png"  />
  
  + 接口设为lan
  
    
  
    <img src="image\接口设为lan.png"  />
  
    <img src="image\添加公钥.png"  />
  
  + 最终实现了ssh远程登录
  
    
  
    <img src="image\远程登录.png"  />
  

---



##  开启 AP 功能

+ 当前待接入 USB 无线网卡的芯片信息可以通过在 Kali 虚拟机中使用 `lsusb` 的方式查看，但默认情况下 OpenWrt 并没有安装对应的软件包，需要通过如下 `opkg` 命令完成软件安装。

  ```
  opkg update && opkg install usbutils
  安装好 usbutils 之后，通过以下 2 个步骤可以确定该无线网卡的驱动是否已经安装好。
  
  # 查看 USB 外设的标识信息
  lsusb
  # Bus 001 Device 005: ID 0cf3:9271 Qualcomm Atheros Communications AR9271 802.11n
  # Bus 001 Device 006: ID 0bda:8187 Realtek Semiconductor Corp. RTL8187 Wireless Adapter
  
  # 查看 USB 外设的驱动加载情况
  lsusb -t
  # /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/12p, 480M
  #     |__ Port 1: Dev 4, If 0, Class=(Defined at Interface level), Driver=, 480M
  #     |__ Port 2: Dev 5, If 0, Class=Vendor Specific Class, Driver=ath9k_htc, 480M
  在上面的命令例子中，芯片名称为 AR9271 的无线网卡已经成功加载了驱动 ath9k_htc ，而另一块芯片名称为 RTL8187 的无线网卡则未加载到匹配的驱动。通过 ifconfig -a 或 ip link 均可以验证系统当前只能识别到一块无线网卡。
  
  通过 opkg find 命令可以快速查找可能包含指定芯片名称的驱动程序包，如下所示：
  
  opkg find kmod-* | grep 8187
  # kmod-rtl8187 - 4.14.131+2017-11-01-10 - Realtek Drivers for RTL818x devices (RTL8187 USB)
  安装 kmod-rtl8187 驱动之后，再执行 lsusb -t 时发现已经成功加载驱动。
  
  |__ Port 1: Dev 6, If 0, Class=(Defined at Interface level), Driver=rtl8187, 480M
  ```

  <img src="image\安装lsusb.png"  />

+ 默认情况下，OpenWrt 只支持 `WEP` 系列过时的无线安全机制。为了让 OpenWrt 支持 `WPA` 系列更安全的无线安全机制，还需要额外安装 2 个软件包：`wpa-supplicant` 和 `hostapd` 。其中 `wpa-supplicant` 提供 WPA 客户端认证，`hostapd` 提供 AP 或 ad-hoc 模式的 WPA 认证。

  <img src="image\安装包.png"  />

 + 通过 `opkg find` 命令可以快速查找可能包含指定芯片名称的驱动程序包（之前的8811AU找不到驱动，换了一个网卡）

   ```
   opkg find kmod-* | grep 8187
   # kmod-rtl8187 - 4.14.131+2017-11-01-10 - Realtek Drivers for RTL818x devices (RTL8187 USB)
   ```

   <img src="image\找驱动.png"  />

+ 安装驱动，由于openwrt网站的问题，经常会安装失败，于是换了一个源，换了一个源又有了新错误，于是换回来了。安装驱动时，还出现了cannot install the packet的错误，于是加了一个参数: **--nodeps**（强制安装）,后成功安装

  <img src="image\成功安装.png"  />

 + 查看 `openwrt` 支持哪些无线网卡驱动可以通过 `opkg find kmod-* | grep wireless` 进行查看。

   

   <img src="image\wireless.png"  />

  + **无线网络在luci界面上显示未成功**（更换网卡试了以后都不行）

   

  

  

  

  

  


