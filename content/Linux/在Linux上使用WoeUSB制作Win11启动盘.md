环境：Debian 12 / 13，WoeUSB-ng，目标：制作Win11安装U盘  
U盘：闪迪 Cruzer Spark 15G（老U盘，中途出现只读警告，但最终制作成功）  
一、前置准备
Win11 ISO镜像（放在~/Downloads/目录）
U盘 ≥8G，本次使用闪迪Cruzer Spark
安装依赖（WoeUSB编译运行所需）
```sudo apt update
sudo apt install git p7zip-full python3-pip python3-wxgtk4.0 grub2-common grub-pc-bin
```
拉取 WoeUSB-ng 源码
```
git clone https://github.com/WoeUSB/WoeUSB-ng.git
cd WoeUSB-ng
sudo pip3 install .
```
二、识别U盘设备（重要！千万别选错硬盘）
```
lsblk
```
本次U盘为 /dev/sdb  
⚠️ 注意：不要写成 /dev/sdb1，要写整块设备 /dev/sdb  
三、制作命令
```
sudo woeusb --device /home/debian/Downloads/SW_DVD9_Win_Pro_11.iso /dev/sdb --target-filesystem NTFS
```
• --device：整块U盘模式
• --target-filesystem NTFS：使用NTFS存放Windows安装文件，支持大于4G单文件
四、过程中遇到的警告 & 坑点
警告：Partition is still detected after wiping all signatures, this indicates that the drive might be locked into readonly mode due to end of lifespan.
◦ 含义：U盘检测到残留分区标记，提示可能进入硬件只读保护
◦ 本次现象：警告弹出，但实际仍然可以写入文件，可以继续等待
◦ 若真的硬件只读：直接更换U盘，无法修复
❗ 禁止操作：不要打开文件管理器挂载U盘分区，挂载会抢占磁盘，导致拷贝卡死、进度停滞
老U盘速度很慢，5.6G镜像耗时很久，耐心等待
五、成功标识
终端出现下面两行代表制作完成：  
Done :)  
The target device should be bootable now  
提示 You may now safely detach the target device 后，拔出U盘。
>更多技术教程宝藏资源请访问达斯沃官网：[www.dasiwo.com](https://www.dasiwo.com)