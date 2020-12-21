# Arch Linux 安装指南

> 前提：已经准备好启动盘(Windows 用rufus， Linux用dd命令)，禁止了电脑的`fast boot`,`secure boot`，完成了其他一些需要修改的东西，并且已经进入Arch的安装界面。

## 一、验证启动模式

```sh
ls /sys/firmware/efi/efivars
```

检查电脑的启动模式是UEFI还是BIOS，如果有这个目录，那么就是UEFI模式。



## 二、连接无线网络

### 1.检查网卡、启用无线网络

```sh
ip link # 列举网络设备及其相关信息
```

查看网络设备信息，找到无线网卡(名字可能会叫`wlo1,wlan0`等，容易找到)，如果无线网卡没有启动的话(已经启动的话可以看到类似`<BROADCAST,MULTICAST,UP,LOWER_UP>`的信息)，打开无线网卡：

```sh
ip link set x up # 这里的x是无线网卡(网络设备)的名字
# iw dev wlp0s4 link    //查看网卡wlp0s4连接状态#
# iw dev wlp0s4 scan    //查看可用热点
```

> 但是如果找不到无线网卡。。。那就通过有线连接上网。

### 2.连接无线网络

因为绝大多数WIFI都是有密码的，为了方便，先生成一个配置文件(针对你要用的那个WIFI)：

```sh
wpa_passphrase wifi_name wifi_password > internet.conf #这个配置文件名字随便起
# 或者 wpa_supplicant -B -i wlps04 -c <(wpa_passphrase "ssid" "passwd")
```

之后：

```sh
wpa_supplicant -c internet.conf -i 无线网卡的名字 & #指定一个网卡连接一个网络，后台运行
```

之后为了我能够动态设置主机的IP，启用`dhcpcd`:

```sh
dhcpcd
```

这样子才算联网成功，其实每次都可以这样上网，而且可以方便应对WIFI连不上的问题。



## 三、更新系统时间

```sh
timedatectl set-ntp true
```



## 四、建立硬盘分区

### 1.查看硬盘情况

```sh
lsblk # 或fdisk -l
```

### 2.分区

```sh
fdisk 硬盘设备名称 #进入对应硬盘，对该硬盘进行分区等操作，之后按需分区
```

### 3.格式化分区

对一般的分区：

```sh
mkfs.ext4 分区名
```

而你要用来挂到载`/boot`或`/boot/efi`的分区则是：

```sh
mkfs.fat -F32 分区名
```

交换分区的话则是：

```sh
mkswap 分区名
swapon 分区名
```

### 4,挂载分区

```sh
mount 分区1 /mnt
mkdir /mnt/home
mount 分区2 /mnt/home
mkdir /mnt/boot
mkdir/mnt/boot/efi
mount 分区3 /mnt/boot/efi
```

个人建议挂引导分区`/boot/efi`，挂`/boot`会找不到启动项(个别电脑)。

> 至此，arch系统的文件结构已经建立好了，就在/mnt下。



## 五、换源

向`/etc/pacman.d/mirrorlist`中增加国内源，如清华源，中科大源等。



## 六、安装基础软件包

使用`pacstrap`装一些基本的软件包：

```sh
pacstrap /mnt base base-devel linux linux-firmware 
```

现在arch的base包中基本的文本编辑器，网络管理工具，man手册什么的，都被去掉了，要自己下，不然，重启后进系统连网都上不了。。。

之后可以装一些必要的软件：

```sh
pacstrap /mnt vi vim # 编辑器
pacstrap /mnt iw wpa_supplicant dialog netctl dhcpcd # 无线网络组件
pacstrap /mnt man-db man-pages #man手册
```



## 七、配置系统

### 1.生成`fstab`文件

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

之后`change root`到新安装好的系统：

```sh
arch-chroot /mnt
```

### 2.设置时区

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime # 设置时区为上海
hwclock --systohc #同步
```

### 3.本地化设置(语言)

```sh
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "zh_CN.UTF-8 UTF-8" >> /etc/locale.gen
#或者去/etc/locale.gen中将en_US.UTF-8 UTF-8和zh_CN.UTF-8 UTF-8的注释去掉
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

### 4.设置主机名

```sh
echo "yourhostname" >> /etc/hostname
```

之后编辑`/etc/hosts`文件:

```
127.0.0.1    localhost.localdomain    localhost
::1          localhost.localdomain    localhost
127.0.1.1    yourhostname.localdomain     yourhostname
```

###  5.设置root密码

```sh
passwd
```

### 6.安装`intel-ucode`或`amd-ucode`

```sh
pacman -S intel-ucode #Intel CPU, amd的话就装amd-ucode
```

### 7.安装引导

对于UEFI系统(一般来说UEFI现在是主流)

```sh
pacman -S grub efibootmgr os-prober
mkdir /boot/grub
grub-mkconfig -o /boot/grub/grub.cfg #生成grub的配置文件
grub-install --target=x86_64-efi --efi-directory=/boot/efi #--bootloader-id=grub --recheck --removable # 安装grub，正常电脑都是x86_64的，通过uname -a查看系统信息就行
```

之后关机重启，不出意外就可以进入arch linux了，后面进行其他配置。



## 八、配置桌面环境

### 1.换国内源

向`/etc/pacman.d/mirrorlist`中增加国内源，如清华源，中科大源等。

> 可以现在加入`archlinuxcn`源，直接在`/etc/pacman.conf`中写入：
>
> ```
> [archlinuxcn]
> Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
> ```
>
> 安装密钥:
>
> ```sh
> sudo pacman -S archlinuxcn-keyring
> ```
>
> 如果报错，需要重新生成一个新的密钥环：
>
> ```sh
> sudo pacman -Syu haveged
> systemctl start haveged
> systemctl enable haveged
> rm -rf /etc/pacman.d/gnupg
> sudo pacman-key --init
> sudo pacman-key --populate archlinux
> sudo pacman-key --populate archlinuxcn
> sudo pacman -S archlinuxcn-keyring
> ```

### 2.安装`xorg`服务

`xorg`服务是Linux桌面的硬件接口，所有的窗口管理器都是x窗口的实现。

```sh
pacman -S xorg xorg-server
```

### 3.安装显卡(核显)

```sh
sudo pacman -S xf86-video-intel  #intel#
sudo pacman -S xf86-video-ati  #amd#
```

### 4.输入设备

```sh
sudo pacman -S xf86-input-libinput
sudo pacman -S xf86-input-synaptics  #触摸板驱动#
```

### 5.安装登录管理器

```sh
sudo pacman -S sddm sddm-kcm
systemctl enable sddm
```

### 6.安装KDE

```sh
sudo pacman -S plasma kde-applications
```

### 7.声音管理器

```sh
sudo pacman -S alsa-utils pulseaudio pulseaudio-alsa
```

### 8.安装输入法

```sh
sudo pacman -S fcitx fcitx-im fcitx-configtool fcitx-cloudpinyin fcitx-googlepinyin kcm-fcitx fcitx-pinyin
```

之后在`~/.bashrc`中加入：

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

### 9.安装Aur助手`yay`

```sh
sudo pacman -S yay
```

### 10.下拉式终端`yakuake`

```sh
yay yakuake # 依赖就会把konsole装上
```

### 11.安装字体

```sh
yay -S wqy-microhei wqy-microhei-lite wqy-zenhei wqy-bitmapfont 
yay -S ttf-dejavu
yay -S noto-fonts-sc
```

基本的配置已经弄好了，后面就按需安装软件就行。

> 有Nvidia独显的，如果头铁可以试着装Nvidia的独显驱动。。。
>
> ```sh
> yay nvidia bbswitch optimus-manager-qt #使用KDE桌面，需安装optimus-manager-qt-kde
> ```
>
> 如果发现驱动崩了，那么重装系统，最后换：
>
> ```sh
> yay nvidia-dkms bbswitch-dkms optimus-manager-qt #使用KDE桌面，需安装optimus-manager-qt-kde
> ```
>
> 如果还崩，那就告别Nvidia独显驱动。

### 12.安装pytorch

```sh
yay -S python-pytorch-cuda #这一项要求/分区的磁盘要足够大，因此最好给个60-70G，整个系统给150G。
```

