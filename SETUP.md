# 安装

## 新的Magento 2项目 (macOS/Linux)

1. 切换新项目的位置（例如 cd ~/Sites/magento2）来创建项目模板，然后运行
	- `curl -s https://raw.githubusercontent.com/mandrills/docker-magento/master/lib/template | bash`

2. 将当前的Magento站点的内容提取到 `src` 文件夹中，或composer命令下载Magento源代码启动新项目
    - `bin/download 2.3.0`

3. 使用自定义域名将条目添加到本地主机文件中。 假设您要设置的域是“ magento2.test”，请输入以下内容。
    - `echo "127.0.0.1 magento2.test" | sudo tee -a /etc/hosts`

4. 使用本地脚本来启动Docker容器：
    - `bin/start`

5. 对于新项目使用以下脚本运行Magento的安装
    - `bin/setup magento2.test`

6. 现在可以在网络浏览器中访问您的网站
    - `open http://magento2.test`

## 新的Magento 2项目 (Windows)

在WSL中与Docker一起使用，遵循完整说明

