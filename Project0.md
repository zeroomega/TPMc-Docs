# 基于TPMc的这次比赛的一些思路和想法 #

##云环境下可信虚拟域管理系统

###作品简介
云计算成为计算系统发展的新趋势。它为人们带来便利的同时也对其安全问题带来了新的挑战。可信计算技术(Trusted Computing)以及这项技术的可信平台模块(TPM)一直以来是解决信息安全问题的一个良好途径。但由于其未针对云计算技术进行设计，使得其在云环境下的应用存在着很多问题。例如:

 1. 基于硬件的TPM只能为一台物理主机服务，无法为在其上运行的虚拟机服务。如果主机损坏，和TPM相关的信息将无法使用。
 2. 基于Xen的虚拟可信平台模块(Virtual Trusted Platform Module, vTPM)可以为虚拟机服务，但使用它必须要求用户能够以特权身份访问物理平台，并且需要管理员为每一台虚拟机进行完整配置。由于云平台上往往运行了大量的虚拟主机，采用传统vTPM将会极大地提高管理和维护成本，同时也对管理员提出了很高的要求。

这些缺陷使得可信计算技术很难在云平台下得到广泛利用。

我们对基于Xen虚拟机的云平台的虚拟可信平台模块(vTPM)进行了修改和改进，使得原有只能验证和保证一台计算机安全的可信平台模块(TPM)，能够应用在多台虚拟机上。同时，虚拟机之间也能跨越物理平台组建一个可信的虚拟域(Trusted Virtual Domain, TVD)，由一个独立的虚拟可信计算模块(vTPM)来保障其安全。 然后，我们的作品为这样的一个平台还设计了一个易用高效的管理平台，使得企业对其应用时的维护成本能够尽可能的降低。

更为具体地说，我们的作品实现了如下功能：

  1. 修改了Xen虚拟机下的虚拟可信平台模块(vTPM)模块，使其能够为多台虚拟机服务，并能允许多台虚拟机跨越多个物理平台来构建可信虚拟域(TVD).
  2. 保证服务可信的同时设计了一套管理系统，能够代替云服务的用户完成可信平台模块(TPM)的根密钥(SRK)的初始化和管理。可以批量地完成多台虚拟机的虚拟可信平台模块(vTPM)的维护工作(证书以及密钥对的管理)以及批量虚拟机(云节点)的创建和维护工作。

我们相信这个作品将能极大地改善可信计算技术在云平台的应用难题，使得更多的企业使用可信计算技术来保障他们的私有云的安全。

###系统环境
<table>
    <tr>
        <th>Product</th>
        <th>Version</th>
    </tr>
    <tr>
        <td>Vmware Workstation</td>
        <td>v9.0.2</td>
    </tr>
</table>


###需要达成的目标以及预期的做法
> 快速创建多个虚拟机

~~Xen允许使用一个模板创建多个类似的虚拟机，因此我们可以事先编写好不同类型的虚拟机模板，使用xm命令对这个模板批量创建同类型的虚拟机。但在实际操作中，遇到了几个暂时没法很好解决的问题。~~

~~虚拟机需要指定内核 (`Kernel`) 以及可用的根文件系统(`Root FileSystem`)才能够正常的工作而即便是最小功能的VM，也不可避免的需要准备一个超过100M的根文件系统镜像。如果选择在创建VM时去复制一个新的根文件系统，将会非常耗时并严重浪费空间。~~

~~最初我想想过使用QEMU Copy-On-Write(QCOW)来解决这个问题，但测试之后发现Xen似乎在这个格式的支持上做了些手脚，让它不那么可靠。之后我则想使用在OpenWRT以及Ubuntu Live CD上大量使用的OverlayFS技术来避免根文件系统镜像的复制，但目前测试用的虚拟机中的Linux内核似乎老得有点过分，这个功能并没有编译进内核。~~

~~暂时这个不是重点，可以暂时先利用复制模板镜像文件应急。~~

Qcow功能已经证实能在Xen 4.1.2中正常使用。虽然和Xen wiki上的配置略有不同，但可以证实如下的方法可以奏效：

>>创建利用RAW文件作为Backing的qcow2文件(假设backing file为base1.img)

\# qemu-img create -f qcow2 qcow.qcow -b /root/img/base1.img

>>然后在配置文件中使用如下配置来加载qcow2文件

disk = [ 'tap:qcow2:/root/img/qcow.qcow,xvda,w' ]

这个方法其实在wiki中已经被提到应该逐步被舍弃，但Xen的开发者很奇怪地在Xen 4.1.2的xl工具中没有将文档里所提到的所有语法全部实现，因此这个方法是唯一可以在Xen 4.1.2下使用qcow2文件格式的方法。

由于qcow2文件并不能作为loopfile在Dom0中被直接挂载，因此在启动创建好的新虚拟机前如果需要修改几个虚拟机中的文件(主要是配置ip地址，hostname等等)就需要用到一些其他的方法：

1.  **xl block-attach**
    
    这是Xen推荐的做法，但很奇怪加载成功后并没有/dev/xvda出现在Dom0中。不过可以通过/proc/device查到设备号自己建立设备文件。

2.  **qemu-nbd**
    
    这是利用nbd(Network Block Device)内核模块，可以用来挂载任何qemu支持的虚拟机磁盘镜像格式。推荐这个做法。

>>在使用这个工具前需要加载nbd模块:

\# modprobe nbd max_part=16

>>然后使用qemu-nbd对文件进行连接:

\# qemu-nbd -c /dev/ndb0 /root/img/qcow.qcow

>>然后使用mount命令进行挂载，即可

\# mount -t auto /dev/ndb0 /mnt/base0

随后可以在/mnt/base0下对虚拟机的镜像进行修改(更改ip ssh配置。写入启动脚本等)，完成后使用

\# umount /mnt/base0

卸载，并使用:

\# qemu-nbd -d /dev/ndb0 

对qcow2文件进行卸载。之后就可以正常启动虚拟机。

> 为新创建的虚拟机分配SSH密钥并初始化vTPM(SRK)

如果说一个管理员获得了新的虚拟机的终端(Terminal)，这两个操作都可以很容易用命令行完成。但在我们的管理系统中，我们希望这个步骤是由应用程序自动完成的，有如下几个方案:
+ 使用/etc/init.d 脚本，写一个只运行一次的脚本去完成SSH和SRK的初始化。但这个做法有缺陷，初始化后的信息是无法告知Dom0的，管理程