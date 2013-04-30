#一些规划中的Python脚本

### createimg.py

该文件的作用是创建指定数量的虚拟机可用镜像，以base0.img为蓝本，新增的镜像名称为client$(num).img  其中$(num)为具体数字，增加后的镜像被写入到sqlite数据库config.db中，同时写入的还包括虚拟机的MAC地址。

使用方法为 
> python2.7 createimg.py -n 5

### setupimg.py

该文件的作用是在Dom0中挂载镜像，在虚拟机启动前设置虚拟机的hostname, ip地址，并生成新的SSH KEY，完成后，将该虚拟机的相关信息写入到config.db中。包括SSH KEY的公钥

使用方法:
> python2.7 setupimg.py -hn "hostname" -ip "192.167.X.X" -gw "192.168.X.1" 

### generatecfg.py

该文件的作用是为某个特定的镜像创建配置文件，使其能够被Xen启动，文件保存于./cfg/下，以domU$(num).cfg的形式命名。

文件名会被写入到config.db中

使用方法:
> python2.7 generatecfg.py -n XX

### loadimgconfig.py

将指定编号的虚拟机的hostname等信息载入到config.db中

使用方法:
> python 2.7 loadimgconfig.py -n XX




