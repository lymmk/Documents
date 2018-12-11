# 写在前面 #
+ 以下所有前期准备，全部基于可编译运行如下项目这一基本前提。
[项目Github地址](https://github.com/facebookresearch/deepmask) 
+ Nvidia Cuda + cudnn

	提供图形计算驱动和深度学习相关算法，图像处理过程中会用到大量GPU运算。
+ Torch + Lua

	Torch7 是一个科学计算框架，支持机器学习算法。提供一个强大的 N 维数组、线性代数程序、神经网络以及以能源为基础模型等。

	Lua是一种轻量级脚本语言，深度学习常用开发语言。
+ Deepmask + Sharpmask

	Deepmask用于图像内容物体识别与分割，Sharpmask为其升级扩展，分割技术更精确。前文提到的开源项目是DeepMask和SharpMask算法的Torch实现。
# 安装Ubuntu #
+ 基于前提条件，最佳系统选择为 ubuntu 14.04 LTS版本。同时由于对GPU运算的需要，不推荐使用虚拟机系统，调取真实显卡困难，且实际运行性能可能也有较大缺陷。
+ 双系统安装这里不详细介绍。注意在磁盘分区时，存放deepmask项目空间要大，至少30-40G左右；“/” 根目录尽量要大，建议最少30-40G空间，安装nvidia cuda的过程中需要大量临时空间。
+ 系统安装完成以后一定要更新软件，注意只是更新软件，不要更新系统。如果不更新，安装显卡驱动可能会出现内核或者gcc错误问题。
# 安装显卡驱动 #
## 检查环境 ##
+ 显示 Nvidia 显卡信息,有显示信息即可。
	>$ lspci | grep -i nvidia  
+ 检查自己的gcc版本
	>$ gcc --version
	>如果没有安装，使用 sudo apt-get install gcc 进行安装
+ 检查是否安装了kernel header
	>$ uname -r
## 下载驱动 [链接](https://www.nvidia.cn/Download/index.aspx?lang=cn)##
+ cuda安装包中也包含驱动程序，单独下载驱动可以选择最匹配的版本。
+ 开始安装
	+ 查看是否启用默认驱动
		>$ lsmod | grep nouveau
	+ 禁用默认驱动
		>$ sudo gedit /etc/modprobe.d/blacklist-nouveau.conf  添加
		
			blacklist nouveau
			options nouveau modeset=0 
		>$ sudo update-initramfs -u 更新下设置
	+ 重启电脑，在登录界面 ctr+alt+f1 进入文本模式
		>$ sudo service lightdm stop 关闭图形界面

		>$ sudo ./Nvidia_*.run

		安装开始，按照提示进行即可。
+ 验证
	>nvidia-smi 可以显示显卡信息即为安装成功
# 安装cuda #
## 下载 ##
+ 根据前提条件决定，最匹配版本 cuda 8.0 [链接](https://developer.nvidia.com/cuda-80-download-archive)
+ 选择对应的系统版本，推荐runfile[local]安装方式
## 开始安装 ##
+ 如前文操作，进入文本模式，关闭图形界面
+ 执行安装命令
	>sudo sh cuda_8.0.44_linux.run -no-opengl-files  必须加后面参数，否则可能会循环进入登录界面
	
	开始安装，第一步是否安装NVIDIA显卡驱动，因为之前单独安装过，选择 no,其它选项都安装，等待完成。
+ 重新启动图形化界面
	>$ sudo service lightdm start
+ 设置环境变量。
	>$ sudo gedit /etc/profile
	
		$ export PATH=/usr/local/cuda-8.0/bin:$PATH
		$ export LD_LIBRARY_PATH=/usr/local/cuda/lib64
+ 验证
	>$ nvcc –V  会输出CUDA的版本信息 ，如果提示没有nvcc命令，按照提示安装nvcc
+ 编译Samples
	+ 安装cuda时指定了samples路径，进入 `NVIDIA_CUDA-8.0_Samples` 目录
	+ 输入 `$ make` 进行编译
	+ 切换到编译后目录
		>cd ~/NVIDIA_CUDA-8.0_Samples/bin/x86_64/linux/release
	+ 输入指令 `$ ./deviceQuery` ，显示结果最后出现 `Result = PASS` 即为成功。
# 安装cudnn #
## 下载 ##
+ 下载cudnn需要登录nvidia开发帐号，登录服务器连接比较困难，根据自己实际网络再想办法吧。Windows系统下安装上NVIDIA显卡驱动会出现一个联网服务，启动方式改为自动启动后登录成功概率会提升很多，DNS改为8.8.8.8也有所帮助。
+ 根据多次编译摸索，cudnn可使用版本在5.1至6.0之间，可在此基础上选择下载。
+ 可下载runtime/dev/samples三个安装包
## 安装 ##
+ 参考官方安装教程  [链接](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html#axzz4qYJp45J2)
# 安装Git #
+ 安装并配置 git
	> $ sudo apt-get install git 

	> $ git config --global user.name "mk"

	> $ git config --global user.email "mk@mk.com"
+ 配置Git SSH
	+ 生成ssh公钥	
		> $ ssh-keygen -t rsa -C "mk@mk.com"
	+ 查看密钥
		> $ cat ~/.ssh/id_rsa.pub 
	+ 复制公钥信息，设置到自己github账户中
		> Settings-->SSH and GPG keys-->New SSH key
# 安装torch #
+ 参考官方文档 [链接](http://torch.ch/docs/getting-started.html#_)
+ 安装需要的依赖库模块
	+ 查看已有的模块列表
		>$ luarocks list 如果没有安装luarocks 使用apt-get安装
	
	+ 确保模块列表包含以下几个模块

			COCO API, image, tds, cjson, nnx, optim, inn, cutorch, cunn, cudnn
		其中一般coco，image,inn三个模块需要手工安装，image/inn 直接使用 luarocks install xxx安装，coco 安装请参考: [COCO](https://github.com/cocodataset/cocoapi)
# 安装deepmask #
+ 参考官方文档 [链接](https://github.com/facebookresearch/deepmask)
# 问题总结 #
主要记录在实际操作中遇到的问题，安装过程中有些网友遇到的问题没有出现，在此没有记录。实际遇到后可自行查找解决办法。
## 运行deepmask项目遇到的问题 ##
+ cannot open </pretrained/deepmask/model.t7> in mode r
	
	一般是因为参数指定的目录文件不存在，或者尝试修改下文件权限。

	网上也有说可以尝试使用LuaJit21，不推荐使用Lua5.+。注意，即便是使用LuaJit,版本太低也可能出现此问题。
+ Failed to load function from bytecode：
	
	网上有说换了编译环境可以解决，看了半天很复杂。我遇到此问题时使用的是Lua5.2，换成LuaJit之后问题解决。
+ module coco not found

	前文有提到，依赖Torch的一个必要模块 coco 未找到，其他几个可以使用 `luarocks install xxx` 进行安装，或者有些是在配置环境是自动安装，COCO比较特殊，需要单独安装，方法见上文。
+ *CocoApi.lua assertion failed*
	
	寻找`json`配置文件，一般出现此问题，是因为找不到该文件，把下载好的`instances_train-val2014.zip`文件解压，然后确认代码中指定到解压后的`json`目标文件

+ CocoApi.lua 145 Expected value but found T_END

	由于 `torch.CharStorage(annFile):string()` size=0 造成。[参考解决办法](https://github.com/torch/torch7/issues/1064)
+ luajit : not enough memeory

	Luajit 32位最高使用 2GB 内存，通过编译64位解释器解决。

	`# make CFLAGS= -DLUAJIT_ENABLE_GC64`
+ cuda runtime error : out of memory

	此时GPU显存不够，硬件配置达不到要求。

+ 其他可能遇到的问题。[供参考](https://software.intel.com/en-us/blogs/2017/08/28/deepmask-installation-problems-and-solutions-3)
## 操作系统相关问题 ##
+ 编译过程中磁盘空间不足

	可以使用 gparted 工具对磁盘进行扩容，详细教程网上有很多。[供参考](https://blog.csdn.net/zhoudengqing/article/details/50474012)
+ BIOS设置，关闭安全启动模式。
+ 新安装的系统一定要先更新下软件，一般开机后会自动弹出更新提示，300MB左右更新包。