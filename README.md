PXE+Kickstart自动化安装
=================================================
步骤解读：
---------------------------------------------------------------------------------------
1、服务器/PC网卡支持PXE，自带有DHCP及TFPT客户端；
2、首先开机后网卡作为DHCP客户端向DHCP服务器请求IP地址，DHCP服务器返回IP地址及PXE文件位置（TFTP中存放的pxelinux.0文件）；
3、网卡作为TFTP客户端向TFTP服务器请求下载pxelinux.0；
4、本地执行pxelinux.0；
5、向TFTP服务器请求并下载配置文件pexlinux.cfg；
6、本地读取配置文件；
7、向TFTP请求并下载vmlinuz文件；
8、向TFTP请求并下载initrd.img文件；
9、启动linux内核；
10、向http服务器请求下载kickstart文件；
11、自动安装系统；
*************************************************************************************
1、yum install httpd createrepo   
用于创建提供本地yum源
2、service httpd start
启动web服务
3、mount /dev/cdrom /mnt
挂载光盘作为本地rpm包来源
4、mkdir /var/www/html/CentOS-6.7
cp -ra /mnt/* /var/www/html/CentOS-6.7/
拷贝光盘镜像文件到WEB服务目录
5、createrepo -pdo /var/www/html/CentOS-6.7/  /var/www/html/CentOS-6.7/
创建本地repo及元数据信息
6、createrepo  -g  $(ls /var/www/html/CentOS-6.7/repodata/*-comps.xml) /var/www/html/CentOS-6.7/
添加软件包group的信息
7、yum install tftp-server dhcp xinetd           xinetd用于管理tftp等软件
8、vi /etcxinetd.d/tftp打开disabled为no
9、service xinetd start
启动xinetd
10、yum install syslinux   需要的依赖文件工具
11、准备相关文件
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
拷贝pxelinux.0文件
cp /mnt/isolinux/* /var/lib/tftpboot            
拷贝initrd.img，vmlinuz等文件
12、cd /var/lib/tftpboot
mkdir pxelinux.cfg
创建pxelinux.cfg目录
13、cp /var/lib/tftpbot/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
将isolinux.cfg文件重命名为default并放在pxelinux.cfg目录下
14、vi /var/lib/tftpboot/pxelinux.cfg/default
找到label linux等四行，在其上方或下方添加
default linux-auto-install
prompt 0
label linux-auto-install
  menu label ^Auto Install System 
  menu default
  kernel vmlinuz
  append initrd=initrd.img ks=http://172.18.81.189/kickstart文件的地址文件名

如果有多张网卡，在append ks=xxxx 这行添加上ksdevice=eth0
15、更改default文件的权限为644
16、将准备好的kickstart文件放在对应的地址（可以使用 system-config-kickstart 图形界面生成）
  示例：victor_disk.cfg
        victor_lvm.cfg
17、cp /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
拷贝dhcp配置文件
18、修改dhcp配置文件
subnet 192.168.106.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.106.101 192.168.106.200;
  option subnet-mask 255.255.255.0 ;
  next-server 192.168.106.10 ;
  filename "pxelinux.0" ;
}
18、启动dhcpd服务
19、配置虚拟机启动安装
 
