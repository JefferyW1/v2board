# 原版 V2board 迁移至 xiao改版
## 本分支支持的后端
 - [修改版V2bX](https://github.com/wyx2685/V2bX)

## 环境要求
Nginx 1.2.2 ☑️ MySQL 5.7 ☑️ PHP 8.2+  ☑️ phpmyadmin-5.2

php必备插件  Install extentions > 
- redis
- fileinfo 
- swoole4
- readline
- event
- inotify (可选，热重载依赖)

PHP 点击Setting > Disabled functions 将 putenv 、 proc_open、 pcntl_alarm 、pcntl_signal 从列表中删除

## 原版迁移步骤

按以下步骤进行面板代码文件迁移：

    git remote set-url origin https://github.com/wyx2685/v2board  
    git checkout master  
    ./update.sh  


按以下步骤配置缓存驱动为redis，然后刷新设置缓存，重启队列:

    sed -i 's/^CACHE_DRIVER=.*/CACHE_DRIVER=redis/' .env
    php artisan config:clear
    php artisan config:cache
    php artisan horizon:terminate

最后进入后台重新保存主题： 主题配置-选择default主题-主题设置-确定保存

## 伪静态+守护任务修改(可选，开启webman必须,不开启可不修改)

```
location /downloads {
}

location / {
try_files $uri $uri/ @backend;
}

location ~ (/config/|/manage/|/webhook|/payment|/order|/theme/) {
try_files $uri $uri/ /index.php$is_args$query_string;
}

location @backend {
proxy_set_header Host $http_host;
proxy_pass http://127.0.0.1:6600;
}

location ~ .*\.(js|css)?$
{
expires 1h;
error_log off;
access_log /dev/null; 
}
```
### webman 守护
```
php -c cli-php.ini webman.php start
```

## 如果开启webman后订阅地址显示为127.0.0.1看下方处理方法
请在nginx内设置加入以下内容
```
proxy_set_header Host            $http_host;
```
###  注意
启用webman后做的任何代码修改都需要重启生效
