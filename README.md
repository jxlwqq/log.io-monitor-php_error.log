# [Log.io](http://logio.org/) 实时监控 php_error.log 日志

## 开启 php_error
实时监控日志的第一步，要首先开启 php_error 的功能。
```bash
vi php.ini
```
修改 PHP 配置文件，将 `;error_log = php_errors.log` 改为 `error_log = /tmp/php_errors.log`，保存后重启 Apache 或者 php-fpm 服务。

## 安装 nodejs

在[官方网站](https://nodejs.org/en/download/)下载 LTS 版本的 nodejs。如果安装最新版本的 nodejs，将会导致 log.io 无法安装。

## 安装 cnpm

使用[淘宝 NPM 镜像](https://npm.taobao.org/)，否则安装过程将会非常缓慢。
```bash
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 安装 log.io

```bash
sudo cnpm install -g log.io --user "your_name" 
```
`your_name` 为用户名，这里必须指定一个用户名，例如 root。因为 log.io 需要在用户的根目录里面建立目录，存放配置信息。

## 启动 log.io 服务

```
log.io-server
```

## 配置 log harvester 信息
```bash
sudo vim ~/.log.io/harvester.conf
```

```smartyconfig
exports.config = {
    nodeName: "application_server",
    logStreams: {
      php: [
        "/tmp/php_errors.log"
      ]
    },
    server: {
      host: '0.0.0.0',
      port: 28777
    }
  } 
```

## 启动 log harvester
```bash
log.io-harvester
```

## 访问 web 界面

URL: [http://localhost:28778](http://localhost:28778)。至此，log.io 已经安装成功，但一旦关闭会话后，`log.io-server` 与 `log.io-harvester` 进程也将会被关闭，所以下一步，我们需要使用 [Supervisor](http://www.supervisord.org/) 来守护这两个进程。


## 安装 supervisor

#### pip 安装
```bash
pip install supervisor
```
#### 生成配置文件
```bash
echo_supervisord_conf > /usr/local/etc/supervisord.conf
cd /usr/local/etc/supervisor.d/
vi logio.ini
```
追加以下内容：

```ini
[program:logio-server]
command=log.io-server
redirect_stderr=true
stdout_logfile=/tmp/logio-server.log


[program:logio-harvester]
command=log.io-harvester
redirect_stderr=true
stdout_logfile=/tmp/logio-harvester.log
```

#### 启动 supervisor
```bash
sudo supervisord -c /usr/local/etc/supervisord.conf
```
#### 查看运行状态
```bash
sudo supervisorctl
```
返回以下结果，则表明配置成功：
```bash
logio-harvester                  RUNNING   pid 7284, uptime 0:17:00
logio-server                     RUNNING   pid 7285, uptime 0:17:00
```

