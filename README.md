# Ubuntu上Docker+Jupyter lab配置

## 1 安装Docker

### 1.1 使用官方安装脚本自动安装

安装命令如下：

		curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

也可以使用国内 daocloud 一键安装命令：

		curl -sSL https://get.daocloud.io/docker | sh 

测试安装是否成功：

		sudo docker run hello-world

### 1.2 免sudo运行docker

查看用户组及成员：

		sudo cat /etc/group | grep docker

若无docker用户组，可以添加docker用户组：

		sudo groupadd docker 

若当前用户不在docker用户组添加用户到docker组 ：

		sudo gpasswd -a ${USER} docker 

增加读写权限：

		sudo chmod a+rw /var/run/docker.sock

重启docker：

		sudo systemctl restart docker 

### 1.3 安装Nvidia Docker

要想使用gpu，还需要安装Nvidia Docker：

		# Add the package repositories
		distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
		curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
		curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
		
		sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
		sudo systemctl restart docker

测试安装是否成功：

		docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi

如果返回正常的nvidia-smi显卡信息则安装成功。

## 2 下载镜像并启动容器

获取官方镜像（可从https://registry.hub.docker.com/查找，以pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime为例）：

		docker pull pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime

启动容器：

		docker run -itd --gpus '"device=0,2,3,4,5,6"' -p 8888:8888 --name jupyter_pytorch1.6.0  -v /home/jingkun:/workspace pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime /bin/bash

其中，

<code>-i</code> 以交互模式运行容器，通常与<code>-t</code>同时使用；

<code>-t</code> 为容器重新分配一个伪输入终端，通常与<code>-i</code>同时使用；

<code>--gpus</code> 指定容器使用的gpu，<code>--gpus all</code>使用所有gpu，<code>--gpus 2</code>使用2个gpu，<code>--gpus '"device=1,2"'</code>指定使用1、2号gpu；

<code>-p</code> 设置端口映射，服务器端口:容器端口;

<code>-name</code> 容器命名，自己随便设置；

<code>-v</code> 挂载目录，容器的可以使用的工作空间，服务器目录:容器目录；

<code>pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime</code> 根据自己使用镜像的名字进行修改。

## 3 安装、配置并运行jupyter lab

### 3.1 进入容器并安装jupyter lab

进入容器：

		docker exec -it jupyter_pytorch1.6.0 /bin/bash

安装jupyter lab：

		conda install jupyterlab

### 3.2 配置jupyter lab

共有两种方法：

（1）

创建配置文件：

		jupyter notebook --generate-config

根据需要修改配置文件，一般建议如下设置（文件尾部添加）：

		c.NotebookApp.allow_remote_access = True
		c.NotebookApp.ip='*'
		c.NotebookApp.open_browser = False
		c.NotebookApp.password = u'yours'
		c.NotebookApp.port =8888
		c.NotebookApp.terminals_enabled = True

c.NotebookApp.password根据自己密码设置，获取方法为在ipython中运行：

		In [1]: from notebook.auth import passwd
		In [2]: passwd()
		Enter password:
		Verify password:
		Out[2]: 'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'

（2）如果之前在服务器配置过，直接复制配置文件也可。

### 3.3 启动jupyter lab

运行jupyter lab：

		jupyter lab

现在就可以在任何机器上访问<code>服务器ip:之前配置的服务器的jupyter lab端口</code>，在docker镜像环境上使用jupyter lab。
