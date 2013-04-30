#TPM Emulator的安装和使用

由于我们的测试环境是没有硬件TPM支持的，因此我们需要使用TPM Emulator去虚拟一个TPM环境。

##安装过程

TPM Emulator的官方网站地址为http://tpm-emulator.berlios.de/

TPM Emulator是一个开源工程，因此需要从源码编译，在Ubuntu 12.04下，需要先下载svn工具:

> \# apt-get install subversion

然后使用svn将TPM Emulator的源码下载下来

> \$ svn checkout svn://svn.berlios.de/tpm-emulator/trunk tpm-emulator

编译TPM Emulator需要GNU MP库以及cmake,因此使用如下命令将这些工具安装:

>\# apt-get install libgmp-dev cmake

使用源码中的build.sh进行编译:

>\# ./build.sh

之后使用make install安装即可:

>\# cd ./build
>\# make install


##使用过程

TPM Emulator的安装脚本略有问题，因此在官方文档上的一些方法可能在我们系统上无法奏效。

TPM Emulator提供了最为底层的设备驱动 tpmd_dev. 并以字符设备文件/dev/tpm的形式用来兼容一些基于TPM的应用程序。假设当前路径为TPM Emulator源码的根目录，则加载该驱动的方法为：
> \# insmod ./build/tpmd_dev/linux/tpmd_dev.ko

但该设备并不提供任何TPM的功能，它仅仅是将相关的TPM请求转发到TPM Emulator的守护进程tpmd上，因此需要手动启动tpmd
> \# tpmd

如果没有任何错误，TPM Emulator就启动成功了。可以尝试使用一些应用来访问该设备。


