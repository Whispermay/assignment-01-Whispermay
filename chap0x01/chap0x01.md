## Linux实验一

#### 实验目的

* 配置无人值守安装`iso`并进行安装

#### 实验环境

* `vbox`虚拟机6.0.10.132072
* `Windows10`宿主机
* `ubuntu-18.04.4-server-amd64.iso`

#### 实验过程

* `VBox`中新建虚拟机，使用`ubuntu-18.04.4-server-amd64.iso`文件

  ![](image\vbox.png)

  * 修改配置文件`01-netcfg.yaml`

    ```
    vim /etc/netplan/01-netcfg.yaml
    ```

    ![](image\01-netcfg.png)

  * 应用修改

    ```
    sudo netplan apply
    ```

    ![](image\ifconfig.png)

* 配置`ssh`远程登陆
  * 下载`putty`
  
  * 虚拟机开启ssh服务
  
    ```
    /etc/init.d/ssh start
    ```
  
    ![](image\sshstart.png)
  
  * 查看虚拟机和宿主机的ip地址
  
    ![](image\ipconfig.png)
  
    ![](image\ifconfig.png)
  
  * 在`putty`中填入虚拟机中与宿主机同一网段的`IP`地址
  
    ![](image\puttyssh.png)
  
* 创建`iso`镜像文件
  * 在当前用户目录（这里是用户名为`cuc`）下创建一个用于挂载`iso`镜像的目录，终端命令为`mkdir loopdir`
  
  * 下载原`ubuntu-18.04.4-server-amd64.iso`镜像文件
  
    ![](image\ubuntu1804iso.png)
  
  * 挂载`iso`镜像文件到`loopdir`目录中
  
    ```
    sudo mount -o loop ubuntu-18.04.4-server-amd64.iso
    ```
  
  * 创建一个`clonecd`目录用于克隆光盘内容
  
    ```
    mkdir clonecd
    ```
  
  * 同步光盘内容到目标工作目录
  
    ```
    rsync -av loopdir/ clonecd
    ```
  
  * 卸载`iso`镜像
  
    ```
    sudo umount loopdir
    ```
  
  * 进入刚刚创建的`clonecd`目录下
  
    * 编辑`isolinux/txt.cfg`，并添加如下内容并保存
  
      ```
      vim isolinux/txt.cfg
      ```
      
      ![](image\txtcfg.png)
  
  * 下载配置完成的`ubuntu-server-autoinstall.seed`(https://github.com/c4pr1c3/LinuxSysAdmin/tree/master/exp)至`preseed`目录下
  
  * 修改`isolinux/isolinux.cfg`将`timeout`改为10
  
    ![](image\isolinuxcfg.png)
  
  * 生成目标定制`iso`
  
    ```
    #安装mkisofs
    sudo apt install mkisofs
    #重新生成md5sum.txt
    #在clonecd目录下
    find . -type f -print0 | xargs -0 md5sum > md5sum.txt
    
    # 封闭改动后的目录到.iso，创建shell文件，将以下内容写入shell文件中
    IMAGE=custom.iso
    
    BUILD=/home/cuc/clonecd/
    
    mkisofs -r -V "Custom Ubuntu Install CD" \
                -cache-inodes \
                -J -l -b isolinux/isolinux.bin \
                -c isolinux/boot.cat -no-emul-boot \
                -boot-load-size 4 -boot-info-table \
                -o $IMAGE $BUILD
    ```
  
    
  
  * 将生成的`custom.iso`镜像下载至宿主机中，在`Windows`命令提示符中输入
  
    ```
    pscp cuc@192.168.56.102:/home/cuc/custom.iso E:custom.iso
    ```
  
    ![](image\customiso.png)
  
    ![](image\pscpiso.png)
  
  * 在`VBox`中新建虚拟机，选择刚刚下载的`custom.iso`文件，完成无人值守安装
  
    

#### 遇到的问题

* 使用`putty`无法远程登录到虚拟机，提示连接失败
  
* 检查发现虽然虚拟机`ip`地址是`192.168.56.1`，也能联网，但是在配置虚拟机网卡的时候指定的`host-only`网卡为另一`192.168.242.1`的网卡，更改后重试成功
  
    ![](image\mistake.png)
  
* 在使用某些命令时提示权限不够
  
* 使用`sudo -i`切换为`root`权限或在命令前加`sudo`临时`root`权限
  
* 远程登陆虚拟机不能以`root`权限登入

#### 参考资料

* https://space.bilibili.com/388851616/channel/detail?cid=103824
* https://www.debian.org/releases/wheezy/example-preseed.txt

* https://github.com/c4pr1c3/LinuxSysAdmin/tree/master/exp

* https://blog.csdn.net/qq_31989521/article/details/58600426