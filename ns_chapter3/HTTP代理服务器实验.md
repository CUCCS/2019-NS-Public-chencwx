#  实验二：HTTP代理服务器实验

## 实验要求

实验验证：在Kali Linux（网关）中安装tinyproxy，然后用主机设置浏览器代理指向tinyproxy建立的HTTP正向代理，在Kali中用wireshark抓包，分析抓包过程，理解HTTP正向代理HTTPS流量的特点。

## 实验环境

+ 实验所需命令行:（根据上课所得整理）

  ```
  - -s 可以设置包的最大长度
  - 增加host-only方便ssh远程操作
  - ps aux | grep apache2 查看apache是否安装
  - curl +victim ip地址
  - apt update &&apt-get install tinyproxy/apache2/
  - tail -F /var/log/apache2/access.log 刷新日志- -s 可以设置包的最大长度
  - 增加host-only的网卡方便ssh远程操作
  - ps aux | grep apache2 查看apache是否安装
  - curl +victim ip地址
  - apt update &&apt-get install tinyproxy/apache2/
  - tail -F /var/log/apache2/access.log 刷新日志
  ```

+ 实验所需工具:
  
  + tinyproxy 
  
    [tinyproxy简介]: http://tinyproxy.github.io/
  
  + apache ：web服务器
  
  + curl:**cURL**是一个利用URL语法在[命令行](https://baike.baidu.com/item/命令行)下工作的文件传输工具

## 实验步骤

### 实验前验证连通性（上次实验所用拓扑环境）

+ 拓扑环境

  ![](https://c4pr1c3.github.io/cuc-ns/chap0x01/attach/chap0x01/media/vb-exp-layout.png)

+ 攻击者无法ping通靶机，靶机可以ping通攻击者

  <img src="image\验证连通性 (2).png" />

+ 攻击者可以上网且可以访问网关

  <img src="image\攻击者可以上网.png" />

  <img src="image\攻击者ping通网关.png" />

+ 网关可以ping通攻击者和靶机

  <img src="image\网关ping通攻击者+靶机.png" />

### 实验过程

+ 在未开启apache前，靶机无法通过浏览器访问攻击者，此时由于web服务器还未开启

  <img src="image\靶机无法访问攻击者（浏览器）.png" />

+ 开启apache服务，命令行是:service apache2 start 

  <img src="image\靶机开启apache.png" />

  > [参考命令行]: https://www.cnblogs.com/kenshinobiy/p/9212344.html

+ 开启apache以后靶机可以通过浏览器访问攻击者

  <img src="image\靶机可以通过浏览器访问攻击者.png" />

+ 攻击者仍无法访问靶机

  <img src="image\靶机无法访问攻击者（浏览器）.png" />

+ 至此，攻击者还是无法访问靶机，接下来使用正向代理，tinyproxy

  + 网关安装tinyproxy,命令行为

    ```
    - apt update && apt-get install tinyproxy //安装tinyproxy
    - sudo apt-get install gedit //为使用gedit命令，即编辑配置文件
    - vi /etc/tinyproxy/tinyproxy.conf  //编辑配置文件
    - service tinyproxy start //开启tinyproxy
    ```

    

    <img src="image\网关安装tinyproxy.png" />

    + 配置文件允许该网段使用本网关为代理，10.0.0.0/8设置为allow，注意:linux中:wq是保存并退出

    <img src="image\配置文件.png" />

    <img src="image\开启tinyproxy.png" />

  + 攻击者配置代理，网关设为代理，端口设为默认值8888

    <img src="image\攻击者设置代理.png" />

    

    + 配置完成后，攻击者在浏览器对靶机进行访问，同时在靶机上抓包

      ```
      命令行:tcpdump -i eth0 -n -s 65535 -w attacker.pcap //抓包长度限制为65535，并把结果保存下来
      ```

      <img src="image\靶机抓包.png" />

      + 攻击者访问靶机的apache，同时让靶机进行抓包，访问两个地址分别为:172.16.111.133和172.16.111.133/nationalday，分别出现apache界面和404

        

        <img src="image\攻击者成功访问靶机.png" />

        <img src="image\攻击者访问404.png" />

      + 查看抓包结果,由于本次实验只关心http协议，所以过滤后剩下以下信息

        <img src="image\抓包概况.png" />

      + 选取其中404的包追踪http流，发现我们能看到代理，但是看不到具体的信息（ip地址等等信息），这个信息可以看到更底层（数据链路层）的数据包

        <img src="image\代理信息.png" />

    ---

    + 攻击者访问https并在网关抓包
    
      <img src="image\访问https抓包.png" />
    
    

## 实验总结

+ 本次实验拓扑：

  <img src="image\拓扑图.png" />

+ 本次实验通过设置代理服务器的方式改变了原来网络的连通性，攻击者在未设置代理之前，无法访问靶机，而在经过tinyproxy的设置后，可以进行访问。

+ 代理服务器可以看到主机访问的网址。

## 实验参考

+ [师姐的作业]: https://github.com/CUCCS/2018-NS-Public-jckling/blob/ns-0x03/ns-0x03/3.md

+ [linux文本编辑]: https://www.cnblogs.com/gaosf/p/10154786.html

+ [网络安全课本]: https://c4pr1c3.github.io/cuc-ns/chap0x03/exp.html

  

​    

## 课后习题

1. 代理技术在网络攻防中的意义？

   (1) 对攻方的意义？

   + 访问本无法访问的机器（本实验中的靶机）
   + 对被攻击者隐藏身份，本实验中靶机无法确定攻击者的身份

    (2) 对守方的意义？

   + 攻击者无法辨认出守方

2. 常规代理技术和高级代理技术的设计思想区别与联系？

   + 高级代理技术的匿名通信技术放⼤了⽹络安全对抗的复杂性 
   + 有时可以通过常用代理技术实现高级代理

​    

​    

​    

​    

​    

​    

​    

​    