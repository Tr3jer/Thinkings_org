---
layout: post
title: Under The Docker Run Kali Linux
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

#### 简介
- `Docker`是一个由GO语言写的程序运行的开源的应用`容器`引擎（Linux containers， LXCs），让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）。几乎没有性能开销,可以很容易地在机器和数据中心中运行。最重要的是,他们不依赖于任何语言、框架包括系统。

- `Kali Linux`是基于Debian的Linux发行版， 设计用于数字取证和渗透测试 和 黑客攻防。
Kali Linux预装了许多渗透测试软件，包括nmap (端口扫描器)、Wireshark (数据包分析器)、John the Ripper (密码破解器),以及Aircrack-ng (一套用于对无线局域网进行渗透测试的软件). 用户可通过硬盘、live CD或live USB运行Kali Linux。Metasploit的Metasploit Framework支持Kali Linux，Metasploit一套针对远程主机进行开发和执行Exploit代码的工具。

#### 好处
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Docker相比KVM之类最明显的特点就是启动快，资源占用小。因此对于构建隔离的标准化的运行环境，轻量级的PaaS， 构建自动化测试和持续集成环境，以及一切可以横向扩展的应用都可轻松应对。具体说来，`Docker在如下几个方面具有较大的优势`：

- 更快速的交付和部署
- 更高效的虚拟化
- 更轻松的迁移和扩展
- 更简单的管理

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kali原镜像所含的渗透套件很丰富，基本渗透中的需求都可以满足，但是长期使用会显得有些臃肿，比如换个环境(迁移)，扔锅(被取证)等等，既然想`方便`/`快捷`/`高效`的使用Kali，Docker虚拟化再合适不过。

#### 安装Docker
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我是在`Mac`下安装的Docker，Docker引擎使用了Linux内核特定的特性，所以要让它运行在OS X上我们需要用一个轻量型的虚拟机，如(Virtualbox, Vmware Fusion, Parallel Desktop)等。用OS X的Docker客户端来控制虚拟Docker来构建，运行以及管理Docker容器。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Boot2Docker是帮助控制虚拟机中 Docker 的工具，它会下载一个安装好docker的虚拟机，并控制其实现docker功能：

	brew install boot2docker

安装Docker client：

	# Get the docker client file
	DIR=$(mktemp -d ${TMPDIR:-/tmp}/dockerdl.XXXXXXX) && \
	curl -f -o $DIR/ld.tgz https://get.docker.io/builds/Darwin/x86_64/docker-latest.tgz && \
	gunzip $DIR/ld.tgz && \
	tar xvf $DIR/ld.tar -C $DIR/ && \
	cp $DIR/usr/local/bin/docker ./docker

	# Set the environment variable for the docker daemon
	export DOCKER_HOST=tcp://127.0.0.1:4243

	# Copy the executable file
	sudo cp docker /usr/local/bin/
	
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;或者直接使用[Boot2Docker.pkg](https://github.com/boot2docker/osx-installer/releases)安装，安装好后运行boot2docker这个client command，这个过程会进行初始化下载一个boot2docer.iso，然后会用ssh生成用于docker的ssh的公钥和私钥对，用于远程.

初始化后运行boot2docker报了个错，对应设置到环境变量即可。

	➜ ~ boot2docker start
	Waiting for VM and Docker daemon to start...
	.o
	Started.
	Writing /Users/CongRong/.boot2docker/certs/boot2docker-vm/ca.pem
	Writing /Users/CongRong/.boot2docker/certs/boot2docker-vm/cert.pem
	Writing /Users/CongRong/.boot2docker/certs/boot2docker-vm/key.pem
	Your environment variables are already set correctly.
	
	To connect the Docker client to the Docker daemon, please set:
    export DOCKER_CERT_PATH=/Users/CongRong/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
    export DOCKER_HOST=tcp://192.168.59.103:2376

	Or run: `eval "$(boot2docker shellinit)"`

#### 安装镜像
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kali-linux-docker的[DockerHub主页](https://hub.docker.com/r/kalilinux/kali-linux-docker/)，或者直接搜索：

	➜ ~ docker search kali
	NAME                                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
	kalilinux/kali-linux-docker               Kali Linux 2.x Base Image                       120                  [OK]
	linuxkonsult/kali-metasploit              Kali base image with metasploit                 26                   [OK]
	linuxkonsult/kali                         Kali Linux 2.0 base image                       8
	brimstone/kali                                                                            3                    [OK]
	kkirsche/kali-linux-docker                Unofficial Kali Linux Docker                    2                    [OK]
	kalilinux/kali-linux-docker-rolling       Kali Linux Rolling Docker Image                 2                    [OK]
	wsec/kali-metasploit                      Official Kali Base image + Metasploit           2                    [OK]
	officialkali/kali                                                                         2
	kalilinux/kali                                                                            2
	lodelestra/kali-linux-metasploit-docker   metasploit docker based on kali-linux-docker    1                    [OK]
	ctarwater/kali-msf                        Kali + Metasploit + Postgresql                  1                    [OK]
	e3rp4y/kali-metasploit                                                                    1                    [OK]
	blackfinsecurity/tha-kali                                                                 1                    [OK]
	andresriancho/w3af-kali                                                                   0                    [OK]
	ctarwater/kali                            Kali base image (no tools)                      0                    [OK]
	netxp/kali                                kali                                            0                    [OK]
	johnsandiford/kali                                                                        0                    [OK]
	miteshshah/kali                           Kali Linux                                      0                    [OK]
	nicot/kali                                                                                0                    [OK]
	lxj616/docker-kali-custom-tools           docker-kali-custom-tools                        0                    [OK]
	jasonchaffee/kali-linux                   Kali Linux Docker Container                     0                    [OK]
	ctarwater/kali-msf-micro                  Kali + Metasploit with all of the Metasplo...   0                    [OK]
	scottj/kali-docker                        Custom Kali 2.0 Docker Build                    0                    [OK]
	digitalshokunin/kali-metasploit                                                           0                    [OK]
	butlerrc30/kali-ssh                       Kali image that generates random cert for ...   0                    [OK]

安装Kali：

	docker pull kalilinux/kali-linux-docker

拉取安装后就可以启动了，因为是纯净的kali镜像所以只有420MB，需要什么直接apt-get。

<img src="http://7xiw31.com1.z0.glb.clouddn.com/3r4tgdfxvc.png">

#### 常见问题
- [After upgrade Boot2Docker 1.5 to 1.6 error append:Failed to get machine “boot2docker-vm”: machine does not exist (Did you run `boot2docker init`?)](http://stackoverflow.com/questions/29799491/after-upgrade-boot2docker-1-5-to-1-6-error-appendfailed-to-get-machine-boot2do/29819016)

- [How to fix “error in run: Failed to get machine ”boot2docker-vm“: machine does not exist”?](http://stackoverflow.com/questions/26572112/how-to-fix-error-in-run-failed-to-get-machine-boot2docker-vm-machine-does-n)

- [FATA[0000] Post http:///var/run/docker.sock/v1.18/images/create?fromImage=kalilinux%2Fkali-linux-docker%3Alatest: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?](http://stackoverflow.com/questions/29294286/fata0000-get-http-var-run-docker-sock-v1-17-version-dial-unix-var-run-doc)