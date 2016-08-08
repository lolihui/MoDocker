# MoDocker

modo动漫私有云（modo.moe）的Docker构建用文件。

您可以基于modo动漫私有云的镜像进行修改和重新分发，但请注明based on modo.moe，thanks。

目前官网已经不可访问。

原帖： [v2ex](https://www.v2ex.com/t/225714?p=2)

视频说明： [土豆](http://www.tudou.com/programs/view/9gS3imQGw6g/)

# 功能说明：

* 免配置，一行代码即可安装完成。
* 使用 Aria2+Yaaw 作为离线工具，支持 BT 、 U 链、 HTTP 、百度云下载。
* 使用 KODExplorer 作为桌面工具，提供 WebOS 级别的文件管理体验，自带的播放器可以在线播放动画，没事你还可以换桌面玩。
* 和文件管理器深度整合的 Modo 漫画浏览器，能对图片进行压缩，方便支持通过浏览器和手机进行浏览。

# 修改
* 使用php7.0，性能更好
* 尽可能删除带有http协议的js/css/xhr资源
* 使用token验证
* token密码不再储存在yaaw.js中，储存于downlaod.html，运行后请及时更改download.html的文件名

# 构建
```bash
git clone https://github.com/huihuimoe/MoDocker
sudo docker build -t modo MoDocker
```

# 运行
```bash
sudo docker run -d -p 6801:80 -p 6800:6800 -v /data/comic:/var/www/html/comic modo
```

或者你也可以使用灰灰构建的image

```bash
sudo docker run -d -p 6801:80 -p 6800:6800 -v /data/comic:/var/www/html/comic huihuimoe/modo
```

然后需要再本机运行nginx，nginx的安装也不再多说……

在nginx中加入如下配置，请按自身条件修改，不需要ssl的去掉ssl即可

```
server {
    listen                       443 ssl http2 spdy; # http2 和 spdy 请注意修改
    server_name                  your_domain;
    server_tokens                off;
    ssl_certificate              /path/to/your_domain.cer;
    ssl_certificate_key          /path/to/your_domain.key;
    ssl_ciphers                  EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers    on;
    ssl_protocols                TLSv1 TLSv1.1 TLSv1.2;
    ssl_stapling                 on;
    ssl_stapling_verify          on;
    ssl_session_cache            shared:SSL:50m;
    ssl_session_timeout          1d;
    # ssl_session_tickets          on;
    # ssl_session_ticket_key       /path/to/ssl/session_ticket.key;
    # ssl_dhparam                  /path/to/dhparams.pem;
    # access_log                   /path/to/your_domain.log;
    location / {
        proxy_set_header         Host $host;
        proxy_max_temp_file_size 128m;
        proxy_pass               http://127.0.0.1:6801/;
    }
    location /jsonrpc {
        proxy_pass               http://127.0.0.1:6800/jsonrpc;
    }
}

server {
    server_name       your_domain;
    server_tokens     off;

    access_log        /dev/null;

    if ($request_method !~ ^(GET|HEAD|POST)$ ) {
        return        444;
    }

    location / {
        rewrite       ^/(.*)$ https://your_domain/$1 permanent;
    }
}
```