---
layout: post
title: Osx Installing Metasploit Framework 
---

{{ page.title }}
================
<p class="date">{{ page.date | date_to_string }} - Tr3jer_CongRong</p>

> 是这样的，为了方便平时使用，就在本地(osx)搭建了msf环境，总结下这个过程还有几个必踩的坑和特殊的坑，坑踩多了就有了此文，谈不上经验。23333

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**首先将msf项目clone下来，等待的这个过程可以安装xcode command line tools，还有几个必要的组件。**

    cd /usr/local/share/
    git clone https://github.com/rapid7/metasploit-framework.git
    xcode-select --install
    brew install autoconf automake libtool pkg-config apple-gcc42 libyaml readline libxml2 libxslt libksba openssl sqlite

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**apple-gcc42安装错误的话这样解决：**

    gcc -v #查看当前gcc
    brew update #更新
    brew tap homebrew/versions
    brew tap homebrew/dupes #增加扩展源
    brew search gcc #搜索brew可安装的gcc
    brew install apple-gcc42 #安装

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**这些都安装成功后进入metasploit-framework/config/目录，更改PostgreSQL数据库配置文件：**

    cp database.yml.example ./database.yml#默认没有database.yml
    vim database.yml
    
    development: &pgsql
    adapter: postgresql
    database: msf
    username: msf
    password: msf
    host: localhost
    port: 5432
    pool: 75
    timeout: 5

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**将配置指令写入到系统环境文件。**

    echo "export MSF_DATABASE_CONFIG=/usr/local/share/metasploit-framework/config/database.yml" >> $HOME/.zshrc
    or
    echo "export MSF_DATABASE_CONFIG=/usr/local/share/metasploit-framework/config/database.yml" >> $HOME/.bash_profile

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**安装PostgreSQL，建议使用brew，还有安装好后的一些操作：**

    brew install postgresql --without-ossp-build
    mkdir -p ~/Library/LauchAgents
    cp /usr/local/Cellar/postgresql/9.5.1/homebrew.mxcl.postgresql.plist ~/Library/LauchAgents/ #确保在boot时自引导启动，这个就解决了启动不成功的坑。
    launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist #启动PostgreSQL service
    initdb /usr/local/var/postgres #初始化postgresql
    createuser -P msf #创建用户
    createdb -O msf -E utf8 msf_db #创建db
    vim ~/.zshrc
    alias pg_start='pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start' #快捷启动
    alias pg_stop='pg_ctl -D /usr/local/var/postgres stop' #快捷关闭

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**如果在创建用户和db时报错，尝试使用brew doctor查看是哪里的问题，通常是因为osx权限的问题。更改权限后重新安装postgresql。**

    sudo chown -R $(whoami):admin /usr/local

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**然后就要安装Ruby环境了，建议使用rbenv，或者brew直接安装Ruby。不！建！议！使！用！RVM！**

    brew install rbenv ruby-build

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**rbenv安装完后，在$HOME/.zshrc或其他配置文件中设置初始化。**

    export PATH="$HOME/.rbenv/bin:$PATH"
    eval "$(rbenv init -)"

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**使用rbenv安装Ruby，根据msf官网的要求进行安装，这个就有点坑了，有时更新完msf后，总是需要指定另一个ruby版本，然后还得重新解决模块包的依赖，忍了。**

    rbenv install --list
    rbenv install 2.3.0

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**安装好后可以在$HOME/.rbenv/versions/下查看所有安装的ruby版本，接下来要初始化ruby。**

    rbenv rehash
    rbenv global 2.3.0
    rbenv local 2.3.0
    rbenv version
    ruby -v

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**说到ruby版本切换，有个坑就是global设置后在msf目录下查看ruby版本还是没有变，使用local就解决了。接下来就是进入msf目录解决模块包依赖。**

    cd /usr/local/share/metasploit-framework
    sudo chmod go+w /etc/zshrc
    sudo gem install pg sqlite3 msgpack activerecord redcarpet rspec simplecov yard bundler
    bundle install
    or
    ARCHFLAGS="-arch x86_64" bundle install

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**如果在安装模块包时出现`Connection reset by peer - SSL_connect`ssl连接错误，修改Gemfile和gem sources指定到http，现在rubygems.org的墙倒了，不然ruby.taobao.org也可以。**

    gem sources -a http://rubygems.org/
    gem sources --remove https://rubygems.org/
    gem sources -l
    *** CURRENT SOURCES ***
    http://rubygems.org
    
    ➜ ~  head -1 /usr/local/share/metasploit-framework/Gemfile
    source 'http://rubygems.org'
    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**如果出现某个模块包安装错误，可以自行使用gem安装替代版本，完成后，删掉`GemFiles.lock`，重新运行`bundle install`，还有就是可能会出现安装pg模块时`Failed to build gem native extension`错误：**

    Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.

    /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby extconf.rb 
    checking for pg_config... yes
    Using config values from /usr/local/bin/pg_config
    checking for libpq-fe.h... yes
    checking for libpq/libpq-fs.h... yes
    checking for pg_config_manual.h... yes
    checking for PQconnectdb() in -lpq... no
    checking for PQconnectdb() in -llibpq... no
    checking for PQconnectdb() in -lms/libpq... no
    Can't find the PostgreSQL client library (libpq)
    *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of necessary
    libraries and/or headers.  Check the mkmf.log file for more details.  You may
    need configuration options.

    ........
    Gem files will remain installed in /var/folders/l7/f3_r_hhs46lfprth_1pw57vw0000gn/T/bundler20150320-66140-15yne3b/pg-0.18.4/gems/pg-0.18.4 for inspection.
    Results logged to /var/folders/l7/f3_r_hhs46lfprth_1pw57vw0000gn/T/bundler20150320-66140-15yne3b/pg-0.18.4/gems/pg-0.18.4/ext/gem_make.out
    An error occurred while installing pg (0.18.4), and Bundler cannot continue.
Make sure that `gem install pg -v '0.18.4'` succeeds before bundling.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**这是pg模块包没有找到本地扩展导致的，解决方法就是安装pg模块包的时候选择上本地扩展，安装成功后重新`bundle install`。**

    gem install pg -- --with-pg-config=/usr/local/bin/pg_config

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**如果碰见`recog`依赖安装错误的话，通常为Ruby版本低的问题，当时rbenv切换到高版本还是不行，索性使用brew直接安装ruby2.3.0就解决了。**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**到这里msf就安装完成了，还可以进入到msf目录将msf*命令ln到bin下。**

    for MSF in $(ls msf*); do ln -s /usr/local/share/metasploit-framework/$MSF /usr/local/bin/$MSF;done

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**启动时出现could not connect to server无法连接Postgresql的情况，删掉/usr/local/var/postgres/postmaster.pid重启postgresql即可解决。**

    psql: could not connect to server: Connection refused
    Is the server running on host "localhost" (127.0.0.1) and accepting
    TCP/IP connections on port 5432?
could not connect to server: Connection refused
    Is the server running on host "localhost" (::1) and accepting
    TCP/IP connections on port 5432?
could not connect to server: Connection refused
    Is the server running on host "localhost" (fe80::1) and accepting
    TCP/IP connections on port 5432?

<img src="https://blog-1252048719.cos.ap-shanghai.myqcloud.com/6udrhfgx.png">