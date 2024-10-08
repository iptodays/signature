# Auto Signature

# 配置文档

---

# 连接服务器

    $ ssh root@ip
    $ mkdir /usr/local/oneinstack && mkdir /usr/local/java

# 服务器环境

    # 需要开启的端口
    3306
    8082
    443
    80

    $ cd /usr/local/oneinstack
    # https://oneinstack.com/install/
    $ yum -y install wget screen
    $ wget http://mirrors.linuxeye.com/oneinstack-full.tar.gz
    $ tar xzf oneinstack-full.tar.gz
    $ cd oneinstack
    $ screen -S oneinstack
    $ ./install.sh

# jdk && resign && shell && html

    # jdk1.8 下载地址
    # 百度云链接:https://pan.baidu.com/s/1h42SC-7GtksqkbSwjocm1g
    # 密码:85r2

    # 将下载的压缩包移动至 /usr/local/java 路径下
    $ cd /usr/local/java && tar zxvf jdk-8u221-linux-x64.tar.gz

    # 下载zsign
    # doc: https://github.com/iizvv/zsign
    $ cd /usr/local/ && git clone https://github.com/iizvv/zsign.git
    $ cd zsign && g++ *.cpp common/*.cpp -lcrypto -O3 -o zsign

    # 使用ssl证书获取 cert-chain.crt && server.crt && server.key
    # 将获取的三个文件移动至 /root 路径下
    # 将 mobileconfig.sh && zsign.sh 移动至 /root 路径下
    $ cd /root/ && chmod +x mobileconfig.sh && chmod +x zsign.sh

    # 解压 html.zip 修改cms&&h5中的index.html文件内容
    # 将修改后的html文件夹移动至服务器根目录下 /

# 环境变量配置

    $ vim /etc/profile

    # 添加环境变量
    # JAVA
    JAVA_HOME=/usr/local/java/jdk1.8.0_221
    CLASSPATH=$JAVA_HOME/lib/
    PATH=$PATH:$JAVA_HOME/bin
    export PATH JAVA_HOME CLASSPATH

    # zsign
    export PATH=/usr/local/zsign:$PATH

    # 更新环境变量
    $ source /etc/profile

# mysql 配置

    $ mysql -uroot -p
    $ GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mysql5.6' WITH GRANT OPTION;
    $ flush privileges;
    $ exit

# nginx 配置

### 将`*.test.com`修改为自己的域名, 并按照自己的 ssl 文件进行填写

    $ cd /usr/local/nginx/conf && mkdir ./cert

    server {
        listen 443 ssl;
        server_name api.test.com;
        charset utf-8;
        ssl_certificate cert/api.test.com.pem;
        ssl_certificate_key cert/api.test.com.key;
        ssl_session_timeout 5m;
        # ssl_protocols  SSLv2 SSLv3 TLSv1;
        # ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_protocols TLSv1.2;
        ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        ssl_prefer_server_ciphers   on;
        location / {
            proxy_pass http://api.test.com;
            add_header 'Access-Control-Allow-Origin' $http_origin always;
                add_header 'Access-Control-Allow-Credentials' 'true' always;
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
                add_header 'Access-Control-Allow-Headers' 'DNT,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range' always;
                add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range' always;
                if ($request_method = "OPTIONS") {
                    return 200;
                }
        }
    }

    server {
        listen 80;
        server_name api.test.com;
        rewrite ^(.*)$ https://$host$1 permanent;
    }

    upstream  api.test.com {
        server  127.0.0.1:8082;
    }

    server {
            listen 443 ssl;
            server_name www.test.com;
            charset utf-8;
            ssl_certificate cert/www.test.com.pem;
            ssl_certificate_key cert/www.test.com.key;
            ssl_session_timeout 5m;
            # ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
            # ssl_protocols SSLv2 SSLv3 TLSv1;
            ssl_protocols TLSv1.2;
            ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
            ssl_prefer_server_ciphers on;
            location / {
                root /html/h5;
                try_files $uri $uri/ /index.html;
            }
        }

    server {
        listen  80;
        server_name  www.test.com;
        rewrite ^(.*)$ https://$host$1 permanent;
    }

    server {
            listen 443 ssl;
            server_name cms.test.com;
            charset utf-8;
            ssl_certificate cert/cms.test.com.pem;
            ssl_certificate_key cert/cms.test.com.key;
            ssl_session_timeout 5m;
            # ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
            # ssl_protocols SSLv2 SSLv3 TLSv1;
            ssl_protocols TLSv1.2;
            ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
            ssl_prefer_server_ciphers on;
            location / {
                root /html/cms;
                try_files $uri $uri/ /index.html;
    			gzip on;
    			gzip_min_length 1k;
    			gzip_buffers 4 16k;
    			gzip_comp_level 2;
    			gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    			gzip_vary off;
    			gzip_disable "MSIE [1-6]\.";
            }
        }

    server {
        listen  80;
        server_name  cms.test.com;
        rewrite ^(.*)$ https://$host$1 permanent;
    }

    # 更新配置
    $ nginx -s reload

# sql 语句

    create schema if not exists signature collate utf8mb4_general_ci;

    use signature;

    create table if not exists apple
    (
        id int auto_increment
            primary key,
        account varchar(255) null comment 'apple开发者帐号',
        count int not null comment '已有设备数量',
        p8 text not null comment '私钥',
        iss text not null comment '在Store Connect上可以点击复制 iss ID',
        kid text not null comment 'your own key ID',
        cerId varchar(255) not null comment '授权证书id',
        bundleIds varchar(255) not null comment '开发者后台的通配证书id',
        create_time timestamp default CURRENT_TIMESTAMP null comment '帐号添加时间',
        p12 varchar(255) null comment 'p12文件地址',
        is_use tinyint(1) default 1 not null comment '帐号是否可用'
    )
    comment '帐号';

    create table if not exists device
    (
        id int auto_increment
            primary key,
        udid varchar(255) not null comment '设备UDID',
        apple_id int not null comment '此设备所使用的帐号id',
        device_id varchar(255) not null comment '设备id',
        is_use tinyint(1) default 1 not null comment '当前设备所处帐号是否可用',
        create_time timestamp default CURRENT_TIMESTAMP null comment '创建时间',
        constraint device_apple_id_fk
            foreign key (apple_id) references apple (id)
    )
    comment '设备';

    create table if not exists user
    (
        id int auto_increment comment '用户id'
            primary key,
        level int default 0 not null comment '等级 0:待管理员审核；1:超级管理员；2:普通用户',
        username varchar(50) not null comment '用户名',
        password varchar(100) not null comment '密码',
        email varchar(100) null comment '邮箱',
        total_device int default 0 not null comment '总设备量',
        use_device int default 0 not null comment '已使用设备量',
        create_time timestamp default CURRENT_TIMESTAMP null comment '创建时间',
        constraint user_email_uindex
            unique (email),
        constraint user_username_uindex
            unique (username)
    )
    comment '用户表';

    create table if not exists package
    (
        id int auto_increment
            primary key,
        name varchar(30) not null comment '包名',
        icon varchar(255) null comment '图标',
        imgs text null comment '预览图',
        user_id int null comment '用户id',
        version varchar(30) null comment '版本',
        bundle_identifier varchar(100) not null comment '安装包id',
        link varchar(100) not null comment '下载地址',
        mobileconfig varchar(255) null comment '获取UDID证书名称',
        download_count int default 0 not null comment '已有下载次数',
        use_device int default 0 not null comment '已使用设备量',
        total_device int default 0 not null comment '总可用设备量',
        build_version varchar(30) not null comment '编译版本号',
        mini_version varchar(30) not null comment '最小支持版本',
        summary text null comment '简介',
        is_stint tinyint(1) default 0 not null comment '是否限制下载',
        create_time timestamp default CURRENT_TIMESTAMP null comment '创建时间',
        sub_title varchar(50) null comment '副标题',
        level float default 4.9 null comment '星级',
        comment_count varchar(50) default '67.8w' null comment '评分数量',
        ranking int default 3 null comment '排行',
        class_name varchar(50) null comment '分类名称',
        age int default 18 null comment '适用年龄',
        size varchar(50) not null comment '文件大小',
        organization varchar(100) not null comment '组织名',
        display varchar(100) not null comment '配置文件标题',
        description varchar(255) not null comment '配置文件描述',
        constraint package_url_uindex
            unique (link),
        constraint package_user_id_fk
            foreign key (user_id) references user (id)
    )
    comment '安装包';

    create table if not exists device_package
    (
        id int auto_increment
            primary key,
        package_id int not null comment 'ipaId',
        device_id int not null comment '设备id',
        is_use tinyint(1) default 1 null comment '设备是否可用',
        create_time timestamp default CURRENT_TIMESTAMP null comment '创建时间',
        constraint device_package_device_id_fk
            foreign key (device_id) references device (id),
        constraint device_package_package_id_fk
            foreign key (package_id) references package (id)
    )
    comment '设备与ipa关联表';

    create table if not exists package_key
    (
        id int auto_increment
            primary key,
        `key` varchar(66) not null comment '密钥',
        is_use tinyint(1) default 1 not null comment '是否可使用',
        package_id int null comment 'packageId',
        use_time datetime null comment '使用时间',
        create_time timestamp default CURRENT_TIMESTAMP null comment '创建时间',
        constraint package_key_key_uindex
            unique (`key`),
        constraint package_key_package_id_fk
            foreign key (package_id) references package (id)
    )
    comment '密钥';

# jar

    # 将jar移动至 /root 路径下
    # 启动jar服务
    $ nohup java -jar app.jar &
    # 查看日志
    $ tail -f nohup.out
    # 查看jar服务使用的pid
    $ lsof -i:8082
    # 停止jar服务
    $ kill -9 pid

