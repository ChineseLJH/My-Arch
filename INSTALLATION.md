# “btw I use Arch”

这是我的第一篇文章，我也只是个彩币，写的不好还请见谅 😢

## 为什么开始尝试 Arch

原先我对于Linux可以说是“浅尝辄止”，使用**Ubuntu**也只是图一时新鲜。真正开始接触Linux还是因为**搭建社团的网站**(现在因为一些原因要重新开始了，所以没法放一个超链接 😭😭😭 )，为了保证本地与服务器环境一致(~~因为当时还不知道docker~~)，我将开发、测试等一切关于网站开发的工作放到了Ubuntu上，随着网站一步步成型，我也逐渐喜欢上Linux的风格(?)。接着就是大数据发力了，给我开始推各种各样的关于Linux发行版的视频，但似乎 **Arch Linux** 是评论区里面最特殊的一个发行版🤔,又碰巧国庆节佳节,我也回了家,没带MCU和SBC,家里正好有一台旧电脑 🤓☝️

## 前期准备

我也是在自己熟悉了一段时间**Ubuntu**的操作之后才开始尝试的 **Arch Linux** ，结合整个的安装过程,我认为是需要有一定的Linux基础的💪，并不是很适合没有尝试过Linux的**纯萌新**来做尝试🤕，所以接下来的教程我会**默认**认为你有一定的基础 👊

对于**Windows**系统下如何创建“未分配”空间以及怎么关闭**BitLocker**，怎么关闭**BIOS**中的安全启动(Secure Boot)等比较常规的操作，我都不会一一教学，接下来就让我们开始吧。

### Arch Linux 镜像下载

[**Arch Linux**](https://archlinux.org/) 这就是Arch Linux的官网,相较于 [**Ubuntu**](https://cn.ubuntu.com/) 的官网，就显得有点凌乱()。[**Download**](https://archlinux.org/download/) 这就是官网的下载网页，我选了全球的geo.mirror.pkgbuild.com的，国内也有很多的镜像可以直接下载，这里就不多赘述。

### Arch Linux 启动盘制作

在启动盘的制作过程中，我选择的是[**Ventoy**](https://www.ventoy.net/cn/index.html)，它一次性可以装**多个镜像** 🤓，并且在平常还能用作普通U盘存放东西，你只需要在他们的官网进行下载，然后解压，接着插上你的U盘，再运行里面的exe文件，但在安装之前，**一定**要先将U盘里的文件、资料转移至本地，然后再进行安装。安装完毕后，你的U盘名字会被改变，你直接将镜像文件放进去就行，但如果你放了**两个及以上**的镜像时，会多出一个很小的分区，这个可以不用管 😎，将你的文件和镜像都放到那个大的分区就行。至此，你的启动盘就**制作完毕**了 🥳🥳🥳

## 开始安装

在这一部分，我会着重来讲 **Arch Linux** 的安装过程以及途中我遇到的错误，在这过程中，b站上的一些视频和各个ai助手帮了我很多忙 😋

### 启动与网络连接

当你成功从U盘启动后，等到出现 **root@archiso ~ #** 后就可以把你U盘**拔**了，因为此时安装程序**完全加载**进内存了。

#### 1.验证启动器

保险起见，先输入 `ls /sys/firmware/efi/efivars`，如果**输出很多**东西，那说明是**UEFI**，这也是我接下来要讲的 😃，但如果没有，那说明是**Legacy BIOS**，虽然和 **UEFI** 有一定出入，但整体区别不大，我的方法可以作一个参考。

#### 2.联网

在安装过程中，需要从网络下载软件包，所以联网这一步必不可少。

+ **有线**：一般直接接上就行。
+ **无线**：我们可以使用`iwctl`工具进行联网。

    ```bash
    # 进入交互式命令行
    iwctl
    # 列出你的无线设备名，例如 wlan0
    device list
    # 扫描网络 (用你的设备名替换 device)
    station device scan
    # 列出扫描到的网络
    station device get-networks
    # 连接到网络 (用你的WiFi名称替换WiFiname)
    station device connect WiFiname
    # 然后是输入密码，完成后推出软件
    exit
    ```

网络连接完成后可以用类似以下的命令来测试网络连接：
```bash
ping bilibili.com
```
看到连续返回类似 `64 bytes from ...` 说明联网成功，按下`Ctrl+C`即可停止`ping`。

#### 3.更新系统时钟

使用以下命令就可以完成系统时钟的更新：
```bash
timedatectl set-ntp true
```

### 磁盘分区与格式化

这是**最关键且最危险**的一步。我们将使用之前在 **Windows** 中创建的“未分配”空间，我们将挂载 **EFI 系统分区 (ESP)** 并创建 **Swap 分区 (交换分区)** 和 **根 (/) 分区** 。

#### 1.确认硬盘名称

在前面就已经提到要在**Windows**下创建“未分配”空间。我们接下来的操作就是建立在**此基础**上的。我们可以用`lsblk`或者`fdisk -l`来看已分区的部分(你在**Windows**下创建的“未分配”空间在这里是看不到的，所以不用慌)，你需要做的就是记住你现在的**EFI 系统分区**，我们接下来会用到它。

#### 2.使用 `cfdisk` 进行分区

使用以下指令进行分区：

```bash
#具体的位置以你之前确认的硬盘名称为主
cfdisk /dev/sda
```

+ **创建 Swap (交换) 分区**
    + 用键盘的 `↑` 和 `↓` ，选中`Free space` 。
    + 用键盘的 `←` 和 `→` ，在**底部菜单**选中`[ New ]` ，然后按下**回车** 。
    + 程序会提示输入 `Partition size:` ，在后面改成你想要的 **Swap分区** 的大小，然后按下**回车** 。
    + 此时一个新分区被创建出来了，确保选中它，然后用用键盘的 `←` 和 `→` ，在**底部菜单**选中`[ Type ]` ，然后按下**回车** 。
    + 在弹出的类型列表中，用键盘的 `↑` 和 `↓` ，选中 `Linux swap` ，然后按下**回车** 。 

+ **创建 Root (根) 分区**
    + 用键盘的 `↑` 和 `↓` ，选中`Free space` 。
    + 用键盘的 `←` 和 `→` ，在**底部菜单**选中`[ New ]` ，然后按下**回车** 。
    + 程序会提示输入 `Partition size:`，这里默认是剩余的**所有空间** ，所以直接按下**回车**就行，当然你也可以自己改动。
    + 这个分区的类型默认就是 `Linux filesystem`，正是我们需要的，不用修改,当然你也可以自行确认。

+ **写入更改并退出**
    +  **检查**屏幕上的分区规划：一个 `Linux swap` 分区，和一个 `Linux filesystem` 分区。
    + **确认无误**后，用 `←` 和 `→`  键选择底部菜单的 `[ Write ]`，按回车。
    + 程序会请求**最终确认** 。输入 `yes` 然后按回车。
    + 最后，选择 `[ Quit ]` 并按回车，退出 `cfdisk` 工具。

至此，分区部分已经完成，分区方案已经正式写入硬盘的分区表中。

#### 3.格式化并启用新分区

分区部分已经结束，现在我们要开始为它们创建文件系统。

**在此之前**，请先运行一次 `lsblk` 命令，确认一下你新建的两个分区的确切名称。根据你的硬盘情况，它们很可能叫 `/dev/sda2` 和 `/dev/sda3` ，或者 `/dev/sda4` 和 `/dev/sda5` 等。下面的指令请务必使用你看到的**正确名称** ！ 我将以 `/dev/sda4 (Swap)` 和 `/dev/sda5 (Root)` **为例**开始操作。

+ **格式化并启用 `Swap` 分区**
    + 用指令 `mkswap /dev/sda4` 进行**格式化** 。
    + 用指令 `swapon /dev/sda4` **激活**改分区。
    
+ **格式化根分区**
    + 用指令 `mkfs.ext4 /dev/sda5` 建立**文件管理系统**，使得分区可以被用来存储目录和文件。

#### 4.挂载分区

**最后一步**，我们将格式化好的分区连接到系统中。

+ **挂载根分区**
    + 用指令 `mount /dev/sda5 /mnt` 将硬盘分区连接到 `/mnt` 。之后所有对 `/mnt` 的写入操作，都是在向你的硬盘写入数据。

+ **挂载 `EFI` 分区**
    + 先用指令 `mkdir -p /mnt/boot/efi` 为挂载 EFI 分区准备一个标准的、空的文件夹。
    + 再用指令 `mount /dev/sda1 /mnt/boot/efi` 将你的 EFI 分区连接到这个刚刚创建的挂载点上。

最后可以在用 `lsblk` 进行检查。

### 安装基本系统

本部分**核心**就是下面的指令，但为了让它运行得**更好** ，我们还需要一些**额外的操作** 。

```bash
pacstrap /mnt base linux linux-firmware nano networkmanager
```

该命令会自动从 **Arch Linux** 的官方软件仓库下载最新版本的**核心**软件包，并将它们解压、安装到你的硬盘分区中。但如果直接运行的话会比较慢，因为默认的下载镜像源再国外，但我们也有办法去国内的镜像源下载。

在运行之前先运行，如果你已经运行了也可以用 `Ctrl+C` 来暂停下载

```bash
nano /etc/pacman.d/mirrorlist
```

在这个文件里面你能看到很多的 `Server = ......` ，你只需要在这些最前面加上下面两行即可。

```bash
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```

这些都是国内的一些镜像源，放在最前面是为了优先使用。

下载过程中难免会遇到 `missing firmware for module:ast xhci_pci_...... qed qla2xxx aic94xx wd719x qla1280 bfa` 之类的，这是因为 `mkinitcpio` 在构建这个微型系统时，会检查你的 `Linux` 内核里所有可能用到的驱动程序（模块）。对于其中一些驱动，它知道这些驱动在某些特定情况下可以加载一个对应的“固件”文件来激活特殊功能或支持特定型号的硬件。于是，它就去检查这个固件文件是否存在。如果不存在，它就会打印出一条 `missing firmware for module: xxx` 的信息。这并不是一个错误 (Error)，而是一个警告 (Warning)。所以对于这些，完全可以**不用在意** 。

### 配置新系统

#### 1.生成 Fstab 文件

首先，我们需要创建一个名为 `fstab` 的文件。它会告诉你的新系统在每次启动时需要挂载哪些硬盘分区。

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

自动化地创建分区挂载表，确保系统启动时能正确找到并挂载自己的根分区 `/` 、`EFI 分区 (/boot/efi)` 等，并启用 `Swap` 分区。这是系统能成功启动的**关键一步** 。
 
 #### 2.Chroot 进入新系统

 这是非常**重要**的一步。执行后，RAM 中的 Live 环境将**切换**到你硬盘上的 `/mnt` 。

 ```bash
 arch-chroot /mnt
 ```

 此时你的**命令行提示符**会有一些变化，可能会变成 `[root@archiso /]#` ，这是正常的。

 #### 3.设置时区

 + **创建时区链接**

    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    ```

    系统通过读取 /etc/localtime 文件来确定自己当前的本地时间。这个命令就是将系统时区设置为中国标准时间。

+ **同步硬件时钟**

    ```bash
    hwclock --systohc
    ```

    这将你在操作系统中设置的时间，写入到电脑主板的 BIOS/UEFI 中。这可以确保 Windows 和 Linux 双系统之间的时间显示不会出错。

#### 4.本地化 / 语言设置

+ **选择要生成的语言环境**

    ```bash
    nano /etc/locale.gen
    ```

    在打开的文件中，用 `PageDown` 或 `↓` 箭头找到下面这两行，并删掉它们行首的 # 号：

    ```bash
    #en_US.UTF-8 UTF-8
    #zh_CN.UTF-8 UTF-8
    ```

    改成

    ```bash
    en_US.UTF-8 UTF-8
    zh_CN.UTF-8 UTF-8
    ```

    然后保存并退出。

+ **生成语言环境**

    ```bash
    locale-gen
    ```

    这个指令可以读取你刚刚修改的 `/etc/locale.gen` 文件，并根据你的选择，实际地在系统里生成对应的语言配置文件。

+ **设置默认语言**

    ```bash
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    ```

    这将创建 `/etc/locale.conf` 文件，并写入 `LANG=en_US.UTF-8` 。这会将整个系统的默认显示语言设置为英文。因为在纯命令行界面（TTY）下，如果没配置好中文字体，所有中文会显示为乱码方块，会给你后续操作带来麻烦。我们通常先设为英文，等后续安装好桌面环境和中文字体后，再在图形界面里再把显示语言切换为中文。

#### 5.设置主机名

```bash
echo "MyArchPC" > /etc/hostname
```

这将你的计算机命名为 `MyArchPC`（你可以换成任何你喜欢的英文名）。这个名字会显示在你的命令行提示符中，也是它在局域网里的名字。

#### 6.设置 `Root` 用户密码

`Root` 用户是 **Linux** 系统中权限最高的超级管理员。必须为它设置一个强密码。

```bash
passwd
```

按下回车后，它会提示你输入新密码 `New password:` ，输完按回车。然后会让你再输入一次 `Retype new password:` 进行确认，再次输入相同的密码后按回车。

#### 7.创建你的个人用户

```bash
useradd -m -G wheel yourusername
```

请将 `yourusername` 换成你想要的用户名，比如 peter lisa 等，**只能**用小写英文字母。

#### 8.为新用户设置密码

和你刚才为 `root` 设置密码一样，现在要为你自己的账户设置一个登录密码。

```bash
passwd yourusername
```

按下回车后，同样会提示你输入两次新密码，输入时屏幕上不会有任何显示。

#### 9.授权新用户使用 `sudo`

+ **安装 `sudo` 软件包**

    ```bash
    pacman -S sudo
    ```
    运行命令后，它会连接网络查找 `sudo` 包，并询问你是否要继续安装 `Proceed with installation? [Y/n]` ，你只需按 `Y` 然后按回车即可。

+ **授权新用户使用 `sudo`**

    ```bash
    EDITOR=nano visudo
    ```

    执行命令后，会用 `nano` 打开 `/etc/sudoers` 文件。

    用 `↓` 箭头向下滚动，找到下面这一行：

    ```bash
    # %wheel ALL=(ALL:ALL) ALL
    ```

    改成

    ```bash
    %wheel ALL=(ALL:ALL) ALL
    ```

    然后保存并退出。

#### 10.安装引导程序 `GRUB`

+ **安装 `GRUB` 及相关工具**

    + 先用 `pacman -S grub efibootmgr os-prober` 安装三个所需软件包。

+ **启用 Windows 检测功能**

    + 出于安全考虑，新版本的 `GRUB` 默认禁用了 `os-prober` 。我们需要手动编辑 `GRUB` 的配置文件来重新启用它。
    + 输入 `nano /etc/default/grub` ,找到 `#GRUB_DISABLE_OS_PROBER=false` 并**删除**前面的‘#’。

+ **将 GRUB 安装到 EFI 分区**

    + 现在，执行真正的安装命令，把 `GRUB` 的引导文件写入我们之前挂载的 `EFI 分区` ，并在主板 UEFI 中注册一个启动项。
    + 用指令 `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ARCH` ,执行后，你应该会看到 `Installation finished. No error reported.` 的提示，表示**安装成功** 。其中 `--bootloader-id=ARCH` 的 **ARCH** ，可以改成任何你想要的，但如果比较长、比较复杂的话，需要加上“”。

+ **生成 GRUB 配置文件**

    + 引导程序已经安装好了，但它还不知道要引导哪些系统。最后一步是让 GRUB 自动扫描你的硬盘，生成包含 **Arch Linux** 和 **Windows** 启动选项的菜单配置文件 `grub.cfg` 。
    + 这个命令在执行时会输出很多信息。请仔细查看它的输出，你应该能看到：
    Found linux image: ... (找到了 **Arch Linux** 的内核),Found initrd image: ... ,Found Windows Boot Manager on /dev/sda1@/efi/Microsoft/Boot/bootmgfw.efi (类似这样的一行，确认它找到了 **Windows** ！)

至此，你的**双系统引导**配置已经**成功**了。

### 收尾与重启

#### 1.启用网络服务

现在我们要让之前安装的 `networkmanager` 做到**开机自启动** ，我们只需要输入下面这一行指令。

```bash
systemctl enable NetworkManager
```

#### 2.退出、卸载分区并重启

现在我们要做的便是**退出新系统**并**重启** 。

+ **退出 `Chroot` 环境**

    + 只需要输入 `exit` ，从你硬盘上的新系统环境，退回到 RAM 的 Live 环境，此时你会发现你的**命令行提示符**变成了原来的样子。

+ **卸载分区**

    + 只需要用 `umount -R /mnt` ,卸载 `/mnt` 目录下的所有挂载点即可。

+ **重启**

    + 此时，你只需要**自信**的输入 `reboot` ，并且**自信**的按下回车，你的 **Arch Linux** 便完成了。

## 结尾

至此，你应该已经完成了对 **Arch Linux** 的手动安装，很高兴你能看到最后，我们有缘再会。