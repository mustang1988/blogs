---
date: 2017-07-12
title: "使用Dockerfile构建自己的docker镜像"
tags:
    - Docker
categories:
    - Docker
gitment: true
---

1. **为什么要自己构建docker镜像?**

    * **docker镜像是啥**

        Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
        
        字面意思就是,docker镜像就是一个可以被docker daemon解析并运行的软件包,里面包含一个完成的操作系统和需要运行的应用程序.

    * **Dockerfile是啥**

        > Docker can build images automatically by reading the instructions from a Dockerfile, a text file that contains all the commands, in order, needed to build a given image.
        
        这是官方文档上给出的描述,Dockerfile包含了一组指令,可以被docker用来自动的读取和执行后得到指定的docker镜像.
        
        Dockerfile是官方提供的一种拥有可读性的,可重复利的,可定制的构建docker镜像的方式.是构建docker镜像的方式之一.
        
        除了Dockerfile还有其他方式来构建docker镜像,但是不能满足下面会提及的一些需求,官方也不是很推荐,就不再介绍了.
        
    * **为啥要自己构建镜像?**

        原因很简单,每个人/每家公司企业都有自己的技术方向和惯用的解决方案,使用通用镜像是无法适配多种多样特殊的要求的.很容易出现诸如性能瓶颈,业务瓶颈等等问题.
    
        为了实现足够高的匹配度,那么自己定制一个针对自己业务/技术进行充分优化和调整的运行环境是必须和必要的.**私人订制**,这就是自己构建镜像的价值所在.
    

2. [**开始构建自己的docker镜像**](https://docs.docker.com/engine/reference/builder/)

    下面将以生产环境所使用的[**php-fpm(v7.1.6)+nginx(v1.12)**](http://10.7.12.3:8080/repo/tag/dev%252Fphp/7.1.6-nginx-opcache)的docker镜像的制作过程来讲解构建自己的docker镜像的过程.

    * **hello world**

        先来一段Dockerfile示例,大致认识下一个Dockerfile的构成:
        
        ```
            FROM        some image
            ENV         some env
            #           some comments
            RUN         some command
            COPY        some files
            EXPOSE      some ports
            WORKDIR     a dir
            VOLUME      a dir
            ENTRYPOINT ["some command","params"]
            
        ```

    * **选择底料**

        ```
            FROM some image
            
        ```
        
        上面示例的这一部分
        
        作用是为你的所要构建的镜像选择一个底包,通常这个底包都会是一个操作系统的官方镜像,比如:
        
        > FROM centos:7.0
        
        这个底包的选择是很随意的,任何一个现成的docker镜像都能被作为你要构建镜像的底包,没有任何限制.
        
        那么一定会有人问,我如果想要自己构建一个操作系统的镜像要怎么玩?我需要一个没有操作系统的底包,去哪找??
        
        答案其实很简单,docker官方提供了最底层的,只包含文件系统的[Base镜像](https://docs.docker.com/glossary/?term=base%20image).
        
        我所选择的底包是php官方构建的[php:7.1.6-fpm-alpine](https://hub.docker.com/_/php/)镜像.这个镜像基于alpine linux的3.4版本制作,特点是体积小,很小,非常小.操作系统本身的大小只有 **不到5MB** (CentOS/Ubuntu的官方底包至少也要300~500MB),加上php官方构建后整体大小不到90MB,非常方便快速的部署.
        
        Dockerfile 示例:
        
        ```
            FROM php:7.1.6-fpm-alpine
        ```

    * **加点调料**

        这一步会对将要构建的环境进行基本的设置,为后续的操作做准备内容包括环境变量的设置,程序依赖的安装,用户/组的创建等等.
        
        > ENV 命令 // 设置环境变量
        
        > RUN 命令 // 执行指定的系统命令
        
        Dockerfile示例:
        
        ```
            ENV NGINX_VERSION 1.12.0
            RUN apk update \
                && apk add --no-cache --virtual .phpdeps \
                $PHPIZE_DEPS \
                openssl-dev \
                pcre-dev \
                libpng-dev \
                freetype-dev \
                libjpeg-turbo-dev \
                libpng-dev \
                libxml2-dev \
                tzdata
        ```

    * **来点硬菜**

        这一步将开始安装所需要的核心应用,nginx
        
        编写方式就像你已经ssh登录进这台主机一样,编译安装等等都可以在这部分进行操作.
        
        Dockerfile示例:
        
        ```
            RUN GPG_KEYS=B0F4253373F8F6F510D42178520A9993A1C052F8 \
            && CONFIG="\
                --prefix=/etc/nginx \
                --sbin-path=/usr/sbin/nginx \
                --modules-path=/usr/lib/nginx/modules \
                --conf-path=/etc/nginx/nginx.conf \
                --error-log-path=/var/log/nginx/error.log \
                --http-log-path=/var/log/nginx/access.log \
                --pid-path=/var/run/nginx.pid \
                --lock-path=/var/run/nginx.lock \
                --http-client-body-temp-path=/var/cache/nginx/client_temp \
                --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
                --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
                --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
                --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
                --user=nginx \
                --group=nginx \
                --with-http_ssl_module \
                --with-http_realip_module \
                --with-http_addition_module \
                --with-http_sub_module \
                --with-http_dav_module \
                --with-http_flv_module \
                --with-http_mp4_module \
                --with-http_gunzip_module \
                --with-http_gzip_static_module \
                --with-http_random_index_module \
                --with-http_secure_link_module \
                --with-http_stub_status_module \
                --with-http_auth_request_module \
                --with-http_xslt_module=dynamic \
                --with-http_image_filter_module=dynamic \
                --with-http_geoip_module=dynamic \
                --with-threads \
                --with-stream \
                --with-stream_ssl_module \
                --with-stream_ssl_preread_module \
                --with-stream_realip_module \
                --with-stream_geoip_module=dynamic \
                --with-http_slice_module \
                --with-mail \
                --with-mail_ssl_module \
                --with-compat \
                --with-file-aio \
                --with-http_v2_module \
                "
        ```
        

    * **餐后甜点**

        这一步主要目的是将一些来自外部的应用/系统的设置,植入镜像中
        
        > COPY 命令 // 从构建机复制指定的文件/目录到容器中
        
        > ADD 命令  // 将指定的文件/目录/URL所对应的文件复制到容器中,和COPY的区别就是支持URL
        
        Dockerfile示例:
        
        ```
            COPY supervisord.conf /etc/supervisord.conf

            COPY start.sh /start.sh
        ```

    * **饭后活动**

        这一步是为了以后使用这个构建出来的环境准备对外暴露的接入方式
        
        > VOLUME 命令 //  创建挂载点,创建的挂载点后,在启动容器时不再需要使用-v参数指定挂在
        
        > EXPOSE 命令 // 用于显示声明容器内应用将会对外暴露的端口,启动容器时需要使用-p参数才会真正映射端口到宿主机
        
        Dockerfile示例:
        
        ```
            EXPOSE 9001 80 443 9000
        ```
        
    * **收工&结束**

        这一步是可选的,用来设置当前制作的镜像启动后的默认动作,在启动容器时可以通过CMD参数来覆盖这个设置的值,不覆盖则默认会执行.
        
        Dockerfile示例:
        
        > ENTRYPOINT 命令 // 设置默认启动行为,只允许有一个,如果写了多个,最后一个会生效,大部分情况下推荐使用ENTRYPOINT,而非CMD命令
        
        > WORKDIR 命令 // 切换操作目录,类似于cd命令,在dockerfile中写cd命令是无效的,需要注意
        
        ```
            ENTRYPOINT ["sh","/start.sh"]
        ```
        
    * **完整Dockerfile奉上**

        [GitHub](https://github.com/mustang1988/docker-images/blob/master/php/7.1.6/with-nginx-tools/Dockerfile)

3. **Tips**

    * **RUN优化**

        dockerfile构建的镜像是分层的,每一层都会占据独立的空间,为了节约,请将能合并的RUN命令,尽可能的合并到一条中用 && 连接.

    * **清理**

        编译安装应用后切记把一些无用的仅编译时需要的组件删除掉,以节约镜像体积,当然还有编译安装时所用到的源码文件,总之就是能删掉的统统删掉,镜像体积能小多少小多少,这和造车轻量化是一样一样一样的.没有人会喜欢一个动不动就几个GB的镜像,那会让你发疯的.

    * **编写说明文档**
    
        编写相关的文档,描述镜像的适用范围,暴露的端口,挂载点等等等,为日后的维护提供支持.
        
    * **持续构建**

        Dockerfile编写完成了,要如何把它构建成镜像呢:
        
        > docker build -t tag名称 Dockerfile所在目录
        
        镜像构建的时间是和Dockerfile文件的复杂程度成正比的,还和其他很多因素有关,比如网速,设备性能等等.
        
        但是每次修改镜像都要手动去执行命令构建是很麻烦的过程,能不能让他监听Dockerfile的改动自动进行呢?答案是可以的.
        
        * docker官方的DockerHub平台就提供了自动构建的功能.下面会简单介绍如何在dockerhub上实现自动构建镜像.
        
            1. 打开[DockerHub](https://hub.docker.com/)页面,**需要翻墙**
            2. 点击登录,推荐使用GitHub帐号登录
            3. 点击右上角的Create->Create Automated Build
    
                ![此处应有图](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-13%2000.15.06.png?imageView/2/w/500)
            
            4. 选择使用GitHub构建
            5. 然后选择你在GitHub创建的构建用的代码库,没有的话可以现去创建一个,公仓私仓都可以(私仓是收费唉,我不是土豪,用不起)
            6. 然后填写一些相关的描述文字,点击创建即可,然后就会进入Build Setting页面
    
                ![此处应有图](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-13%2000.18.06.png?imageView/2/w/500)
            
            7. 记得勾选左上角的"When active, builds will happen automatically on pushes.",然后选择代码库指定分支,指定目录和对应的镜像Tag,点击保存即可
            8. 完成上述设置后,当你向指定分支触发push代码的操作后,就会自动读取该分支,该目录下的Dockerfile自动构建镜像了,需要注意构建文件必须使用固定的文件名Dockerfile,大小写敏感.

        * 由于DockerHub需要翻墙,对吾等天朝民工太不友好,阿里,网易均提供了类似的仓库和构建服务.经过构建速度/镜像拉取速度等方面的比对,发现阿里提供的构建服务完爆网易蜂巢,还是很推荐的.

            1. 打开[阿里开发者平台](https://dev.aliyun.com/search.html?spm=5176.1972344.0.1.8Ee7SL)
            2. 点击右上角的管理中心,并登录
            3. 创建一个镜像仓库

                ![此处应有图](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-13%2000.46.06.png?imageView/2/w/500)
            
            4. 设置代码源位GitHub,当然也支持其他私有的git仓库
            5. 然后同DockerHub类似的设置分支,目录,标签名等参数(阿里支持非Dockerfile命名的构建文件,这个很贴心,给个赞)

                ![此处应有图](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-13%2000.26.25.png?imageView/2/w/500)
            
            6. 推送你的Dockerfile到仓库吧
            7. 记得勾选设置中的"代码变更时自动构建镜像"和"海外机器构建",你会发现下载海外依赖包的速度真的是飞快,比DockerHub还要快一倍以上,赞~赞~赞~
            8. 在仓库基本设置中可以看到自己构建镜像的公开地址,直接复制后面接上":tag名称"就可以拉取了.
            
                ![此处应有图](http://oojbdbtdp.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-13%2000.29.02.png?imageView/2/w/500)
                
        * 使用自动构建时,记得把你需要ADD或者COPY进镜像的文件和Dockerfile一起提交到版本库.
