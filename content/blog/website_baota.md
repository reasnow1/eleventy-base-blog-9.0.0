---
title: 宝塔面板设立网站 Baota Panel website
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-12
tags: 网站Website
---

宝塔面板设立nextcloud

首先重开，选择宝塔面板

弄完后，打开防火墙

为安全，可以只开需要进行操作的电脑的IP。


![baota_ip]({% staticBase %}{% endstaticBase %}/website/baota_ip.png)

申请SSL：需要加一个验证

![baota_ssl]({% staticBase %}{% endstaticBase %}/website/baota_ssl.png)

然后证书选择pem和key的形式。

进入宝塔，进去可以选改一下密码，加一下SSL，改端口

![baota_setup]({% staticBase %}{% endstaticBase %}/website/baota_init_setup.png)

然后安装docker

![baota_docker]({% staticBase %}{% endstaticBase %}/website/baota_docker.png)

注意：先数据库，然后nextcloud，nextcloud的数据库选择刚建的这个。两个都不用打开外部访问。

因为是127.0.0.1来访问的。

在网站这里安装nginx之后弄反向代理

![baota_proxy1]({% staticBase %}{% endstaticBase %}/website/baota_proxy1.png)
![baota_proxy2]({% staticBase %}{% endstaticBase %}/website/baota_proxy2.png)
![baota_proxy3]({% staticBase %}{% endstaticBase %}/website/baota_proxy3.png)

这样，就是访问https://kdxc.uno:端口 来进入网盘。

改config：

![baota_config1]({% staticBase %}{% endstaticBase %}/website/baota_config1.png)
![baota_config2]({% staticBase %}{% endstaticBase %}/website/baota_config2.png)
![baota_config3]({% staticBase %}{% endstaticBase %}/website/baota_config3.png)


增加：
```php
'overwrite.cli.url' => 'https://kdxc.uno:端口',
  'overwriteprotocol' => 'https',
  'trusted_proxies' => 
  array (
    0 => 'IPV4网关',
  ),
```

![baota_config4]({% staticBase %}{% endstaticBase %}/website/baota_config4.png)

即可用APP登录。（注：可能要重启容器。）

![baota_nextcloud_login]({% staticBase %}{% endstaticBase %}/website/baota_nextcloud_login.png)

点APPS

![baota_nextcloud_apps]({% staticBase %}{% endstaticBase %}/website/baota_nextcloud_apps.png)

这里把TOTF Auth打开。由于我已经开了，这里没看到了。

然后在这里：

![baota_nextcloud_totf]({% staticBase %}{% endstaticBase %}/website/baota_nextcloud_totf.png)

即可用验证器。

右上点USERS加用户。

差不多这样

另：多加网页：

![baota_add_webpage]({% staticBase %}{% endstaticBase %}/website/baota_add_webpage.png)

```json
server {
    listen 65432 ssl;                     # 开启 SSL
    server_name kdxc.uno;                # 与证书域名匹配

    ssl_certificate /......;
    ssl_certificate_key /......;
    # 可复用原 SSL 配置，也可以从原 server 块复制更多 SSL 相关设置

    root /www/wwwroot;
    index index.html;

    location / {
    
        #资源访问
    # 添加 CORS 头
        # 允许你的慢服务器域名访问（推荐）
    add_header 'Access-Control-Allow-Origin' 'http://www.tongda.xyz' always;
    # 如果想让任何网站都能用（简单但安全性稍低），可以换成 *
    # add_header 'Access-Control-Allow-Origin' '*' always;

    # 可选：明确允许的方法和头
    add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range' always;
    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range' always;

    # 处理预检请求（OPTIONS）
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
    }
    
    
        try_files $uri $uri/ =404;
    }
    
        # 精确匹配 /tower/ 路径（注意末尾斜杠）
        # 魔塔游戏提交分数重定向
    location = /tower/ {
        return 301 http://www.tongda.xyz/11ty_blog/blog/website_tower_score/;
    }

}
```

补充：添加备案号：

在[nextcloud根目录]/core/templates/layout.guest.php

```html
		<footer class="guest-box <?php if ($longFooter === '') {
			p('hidden');
		} ?>">
			<p class="info">
				<?php print_unescaped($longFooter); ?>
			</p>
			<a href="https://beian.miit.gov.cn/#/Integrated/index" target="_blank" rel="noopener noreferrer">
            桂ICP备2025073747号-1 📜
            </a>
            <a href="https://beian.mps.gov.cn" target="_blank" rel="noopener noreferrer">
            桂公网安备45132202000154号
            </a>
		</footer>
```