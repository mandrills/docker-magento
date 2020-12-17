<h1 align="center">mandrills/docker-magento</h1>



## 目录

- [安装](#安装)
- [升级](#升级)
- [自定义CLI命令](#自定义CLI命令)
- [其他](#其他)


## 安装

### 自动安装 (新项目)

从你安装的目录运行下面这一行命令即可完成项目安装

#### 无示例数据

```bash
curl -s https://raw.githubusercontent.com/mandrills/docker-magento/master/lib/onelinesetup | bash -s -- magento2.test 2.3.5
```

#### 有示例数据

```bash
curl -s https://raw.githubusercontent.com/mandrills/docker-magento/master/lib/onelinesetup | bash -s -- magento2.test with-samples-2.3.5
```

上面的 `magento2.test` 定义了要使用的域名，而 `2.4.1` 定义了要安装的Magento版本。 请注意，由于我们需要对 `/etc/hosts` 进行写操作以进行DNS解析，因此在设置过程中会提示您输入系统密码。

如果你想与Magento一起自动安装示例数据，请在版本前加上 `with-samples-` 。

在上述一行命令完成运行后，你就可以通过`https://magento2.test`访问您的站点。

### 手动安装

和上面运行的命令结果一致，只需将 `magento2.test` 替换成你设置的域名。

#### 新项目

```bash
# 下载Docker Compose模板:
curl -s https://raw.githubusercontent.com/mandrills/docker-magento/master/lib/template | bash

# 下载使用的Magento版本:
bin/download 2.4.1

# 如果下载失败，脚本将尝试使用Composer下载Magento

# 或者可以直接使用Composer安装, 运行:
#
# 开源版(Open Source):
#
# rm -rf src
# composer create-project --repository=https://repo.magento.com/ --ignore-platform-reqs --prefer-dist magento/project-community-edition=2.4.1 src
#
# 商业版(Commerce):
#
# rm -rf src
# composer create-project --repository=https://repo.magento.com/ --ignore-platform-reqs --prefer-dist magento/project-enterprise-edition=2.4.1 src

# 创建站点主机名:
echo "127.0.0.1 ::1 magento2.test" | sudo tee -a /etc/hosts

# 运行Magento的安装命令:
bin/setup magento2.test

# 访问站点
open https://magento2.test
```

#### 现有项目

```bash
# 下载Docker Compose模板:
curl -s https://raw.githubusercontent.com/mandrills/docker-magento/master/lib/template | bash

# 替换为现有的Magento源代码:
cp -R ~/Sites/existing src
# 或者: git clone git@github.com:myrepo.git src

# 创建站点主机名:
echo "127.0.0.1 ::1 yoursite.test" | sudo tee -a /etc/hosts

# 启动容器命令，将文件复制到里面然后重新启动容器:
docker-compose up -d
rm -rf src/vendor
bin/copytocontainer --all ## 初始化拷贝可能需要几分钟...

# 安装composer依赖项, 然后用命令将vendor复制到主机 (便于本地调试):
bin/composer install
bin/copyfromcontainer vendor

# 导入已有的数据库备份:
bin/mysql < backups/magento.sql

# 使用Docker更新数据库凭证:
# 注意: MySQL凭证是从 env/db.env 启动时定义的。
# vi src/app/etc/env.php

# 导入项目的特定环境设置:
bin/magento app:config:import

# 将原有 URL 设置成本地环境 URL (如果没有在env.php里定义):
bin/magento config:set web/secure/base_url https://xxxxxx.test/
bin/magento config:set web/unsecure/base_url https://xxxxxx.test/

bin/restart

# 访问站点
open https://magento2.test
```

> For more details on how everything works, see the extended [setup readme](https://github.com/markshust/docker-magento/blob/master/SETUP.md).

## 升级

要将项目环境 `docker-magento`更新为最新版本, 运行:

```
bin/update
```
建议将docker config文件保留在版本控制中，以便可以在更新后监视文件的更改。 在检查了代码更新并确保它们按预期更新后，运行 `bin/restart` 重新启动容器以使新配置生效。

建议将Docker配置文件保存在一个库中，将Magento代码放在在另一个库里。 这样做的目的是确保Magento路径位于一个特定库的上游，这使自动构建管道和部署易于管理，并保持与Magento Cloud等项目的兼容性。


## 自定义CLI命令

- `bin/bash`: docker容器的bash提示符，主要用于访问docker中的文件系统。
- `bin/cli`: 运行任何CLI命令而无需进入bash提示符。 例如 `bin/cli ls`
- `bin/clinotty`: 在没有TTY的情况下运行任何CLI命令。 例如 `bin/clinotty chmod u+x bin/magento`
- `bin/composer`: 运行composer命令 例如 `bin/composer install`
- `bin/copyfromcontainer`: 将文件夹或文件从容器复制到主机。 例如 `bin/copyfromcontainer vendor`
- `bin/copytocontainer`: 将文件夹或文件从主机复制到容器。 例如 `bin/copytocontainer --all`
- `bin/dev-urn-catalog-generate`: 为PHPStorm生成URN并将路径重新映射到本地主机。 运行此命令后重新启动PHPStorm。
- `bin/devconsole`: `bin/n98-magerun2 dev:console` 命令的别名
- `bin/download`: 下载特定的Magento版本并将其解压缩到src目录中。 如果存档下载失败，它将尝试使用Composer下载。 例如 `bin/download 2.3.5`。
- `bin/fixowns`: 修复容器中的文件系统所有权。
- `bin/fixperms`: 修复容器中的文件系统权限。
- `bin/grunt`: 运行grunt命令。 例如 `bin/grunt exec`
- `bin/magento`: 运行Magento Cli命令。例如 `bin/magento cache:flush`
- `bin/mysql`: 通过数据库配置 `env/db.env` 运行命令。例如 `bin/mysql -e "EXPLAIN core_config_data"` or`bin/mysql < backups/magento.sql`
- `bin/mysqldump`: 备份Magento数据库。例如 `bin/mysqldump > backups/magento.sql`
- `bin/n98-magerun2`: 访问 n98-magerun 命令。例如 `bin/n98-magerun2 dev:console`
- `bin/node`: 运行node命令。例如 `bin/node --version`
- `bin/npm`: 运行npm命令。例如 `bin/npm install`
- `bin/pwa-studio`: (Bate)启动PWA Studio服务器。 请注意Chrome会抛出SSL证书错误并且不允许您查看该网站，但Firefox会。
- `bin/redis`: 从redis容器运行命令。 例如 `bin/redis redis-cli monitor`
- `bin/remove`: 删除所有容器
- `bin/removeall`: 删除所有容器，网络，卷和映像。
- `bin/removevolumes`: 删除所有卷。
- `bin/restart`: 重新启动所有容器。
- `bin/root`: 以root用户身份运行任何CLI命令，而无需进入bash提示符。 例如 `bin/root apt-get install nano`
- `bin/rootnotty`: 以root身份运行任何CLI命令（不带TTY）。 例如 `bin/rootnotty chown -R app:app /var/www/html`
- `bin/setup`: 从源代码（带有可选域名）安装Magento。 默认为`magento2.test`。 例如 `bin/setup magento2.test`
- `bin/setup-grunt`: 安装和配置Grunt任务运行程序以编译less文件
- `bin/setup-pwa-studio`: 安装PWA Studio(需要在主机上安装NodeJS和Yarn) 传送到站点，否则默认使用 `master-7rqtwti-mfwmkrjfqvbjk.us-4.magentosite.cloud` 。 例如 `bin/setup-pwa-studio magento2.test`
- `bin/setup-ssl`: 为一个或多个域生成SSL证书。 例如 `bin/setup-ssl magento2.test magento3.test`
- `bin/setup-ssl-ca`: 生成证书CA并将其复制到主机。
- `bin/start`: 启动所有容器。
- `bin/status`: 检查容器状态。
- `bin/stop`: 停止所有容器。
- `bin/update`: 将项目环境`docker-magento`更新为最新版本。
- `bin/xdebug`: 禁用或启用Xdebug。 允许 `disable` (默认) 或者 `enable` 参数 例如 `bin/xdebug enable`

## 其他

### 数据库

每个服务的主机名是 `docker-compose.yml`文件中服务的名称。 因此，例如，从Docker容器访问MySQL时，其主机名是 `db`（而不是 `localhost`）。 Elasticsearch的主机名为 `elasticsearch`。

连接到Docker实例的MySQL CLI工具，请运行:

```
bin/mysql
```

您可以使用 `bin/mysql` 脚本导入数据库，例如，存储在本地主机目录中的文件`backups/magento.sql`。

```
bin/mysql < backups/magento.sql
```
您也可以使用 `bin/mysqldump` 脚本导出数据库。 该文件将保存在本地主机目录中的`backups/magento.sql`中：

```
bin/mysqldump > backups/magento.sql
```

### Composer 验证

首次安装Magento Marketplace需要验证身份.

复制 `src/auth.json.sample` 到 `src/auth.json`. 然后分别用公钥和私钥更新用户名和密码值，最后通过 `bin/copytocontainer auth.json` 同步到容器中。

### 邮件(Mailhog)

通过访问 [http://127.0.0.1:8025](http://127.0.0.1:8025) 查看在本地发送的电邮。


### Redis

Redis现在是默认的缓存和会话存储引擎，并且在新安装中运行 `bin/setup` 时会自动配置和启用。

使用以下几行命令在现有安装上启用Redis:

**启用缓存:**

`bin/magento config:set --cache-backend=redis --cache-backend-redis-server=redis --cache-backend-redis-db=0`

**启用全页缓存:**

`bin/magento config:set --page-cache=redis --page-cache-redis-server=redis --page-cache-redis-db=1`

**启用会话:**

`bin/magento config:set --session-save=redis --session-save-redis-host=redis --session-save-redis-log-level=4 --session-save-redis-db=2`

你也可以通过运行命令来监视Redis: `bin/redis redis-cli monitor`

### Linux

如果使用了Elasticsearch，你可能必须增加主机系统上的虚拟内存映射数。添加一行代码到 `/etc/sysctl.conf`:

```
vm.max_map_count=262144
```

### Blackfire.io

These docker images have built-in support for Blackfire.io. To use it, first register your server ID and token with the Blackfire agent:

Docker映像已经内置支持Blackfire.io。 要使用它请先在Blackfire中注册您的服务器ID和令牌

```
bin/root blackfire-agent --register --server-id={YOUR_SERVER_ID} --server-token={YOUR_SERVER_TOKEN}
```

接下来，打开 `bin/start` 脚本并取消该行的注释:

```
#bin/root /etc/init.d/blackfire-agent start
```

最后，用`bin/restart`重新启动容器。 这样做之后，所有内容都已配置完毕，你可以使用Blackfire浏览器扩展对Magento商店进行配置。

