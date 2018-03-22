## 前言

前不久把网站升级到了`HTTPS`。
看了下阿里云的证书价格，望而生畏。于是搜了一下免费的`SSL`证书，果然，[Let's
Encrypt](https://letsencrypt.org/)这个平台免费提供三个月的证书。于是乎参考[Let's Encrypt，免费好用的
HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html)进行折腾，但是始终卡在

> 找不到 /etc/ssl/openssl.cnf 文件

这一步，于是换了官方工具`Certbot`执行，验证成功。

## 开始

### 下载

首先需要下载`certbot-auto`这个脚本，并让这个脚本可以被所有用户组用户执行:



    wget https://dl.eff.org/certbot-auto
    chmod a+x ./certbot-auto

### 生成证书

接着我们执行生成证书的指令:



    ./certbot-auto certonly --webroot -w /data/wwwroot/typecho -d www.hellonine.top

`-w`对应的是我们的站点目录，`-d`对应的是我们的域名。

我的`Linux`是`CentOS6.5`，默认安装的是`Python2`，所以在执行的过程中会发现必须要升级到`Python3`，因此需要先把`Python`升级并且更新`/usr/bin`下面的`Python`软链:



    wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz
    tar zxvf Python-3.6.0.tgz && cd Python-3.6.0
    ./configure
    make all && make install && make clean

为了防止出错，我们需要把原来的做一个备份:



    mv /usr/bin/python /usr/bin/python2.6.6
    ln -s /usr/local/bin/python3.6 /usr/bin/python
    python -V

此时我们会发现，`python`的软链已经完成。不过为了兼容之前`python2`安装的一些软件，我们需要修改我们的`yum`:



    vi /usr/bin/yum

将`#!/usr/bin/python`修改为`#!/usr/bin/python2.6.6`

这个时候我们再去执行之前的指令，又会发现另外一个错误，告诉我们需要安装一个`urllib2`的包，于是执行:



    pip install urllib2

此时，我们的证书已经下载完毕，可以在`/etc/letsencrypt/live/$your_domain`下面进行查看。

### 修改nginx

此时，我们在`nginx`中设置我们已经下载好的公钥和私钥的地址即可:



    ssl_certificate      /etc/letsencrypt/live/$your_domain/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/$your_domain/privkey.pem;

如果你愿意，也可以让`HTTP`重定向到`HTTPS`:



    server {
            listen 80;
            server_name www.hellonine.top hellonine.top;
            return 301 https://$server_name$request_uri;
    }

### 定期更新脚本

由于证书90天过期，所以我们需要有一个`crontab`来完成我们的定期更新任务:



    30 1 * */2 6 /your_path/certbot-auto renew --quiet --no-self-upgrade > /dev/null 2>&1

其实我比较想聊一下这个`> /dev/null 2>&1`指令，而这个指令也是我们经常会看到的一个指令，他的意思用一句话概括就是:

> 丢弃所有的标准输出和标准错误

#### >

`>`指令非常常见，就是我们在`shell`中的重定向，这里不多做赘述。

#### /dev/null

`/dev/null`代表一个会被丢失的“黑洞”，即我们无法找到输出的内容。

#### 2>&1

在`shell`中，0代表标准输入，1代表标准输出，2代表标准错误，其实这些数值我们也经常在一些语言作为预定义常量见到，比如PHP。
而中间的`&`代表二者使用同一个文件描述符，让二者在同一个地方输出。

## 参考

[HTTPS 简介及使用官方工具 Certbot 配置 Let’s Encrypt SSL 安全证书详细教程](https://linuxstory.org
/deploy-lets-encrypt-ssl-certificate-with-certbot/)
[shell中>/dev/null
2>&1是什么鬼？](http://www.kissyu.org/2016/12/25/shell%E4%B8%AD%3E%20:dev:null%202%20%3E%20&1%E6%98%AF%E4%BB%80%E4%B9%88%E9%AC%BC%EF%BC%9F/)
