## 基本

### Agent
Agent就是一个步骤的代理
抓取数据源、处理数据、输出为RS

+ Website Agent 解析网页、XML 文档和 json 数据，最常使用
+ Event Formatting Agent 事件信息格式化，可以对收到的信息内容进行格式化，允许添加自定义新内容
+ Phantom Js Cloud Agent 借助 Phantom 抓取动态页面源码，防止部门网站屏蔽爬虫
+ Trigger Agent 监控事件反馈信息的触发器，多用来过滤部分内容
+ De Duplicate Agent 去重
+ Data Output Agent 将数据以 RSS 和 Json 的形式向外部推送
+ Liquid Output Agent 自定义格式数据输出，可以用它创建 HTML 页面，json 数据等
+ Webhook Agent
+ Trigger Agent 监测敏感事件，然后可以用来发送邮件等提醒。
+ Javascript Agent 允许执行自定义的 JS 代码，可以用于个性化操作
+ Digest Agent 汇总节点，收集所有收到的事件再作为一个事件发送出去
+ Email Agent 用邮箱发送最新接收到的讯息
+ Post Agent 可以由其他节点触发，根据固定模板合并事件信息，并以 POST 或 GET 方式向指定的 URL 发起请求
+ Delay Agent 可以作为事件或者副本的暂存器或者缓冲区，统一触发发布
+ Scheduler Agent 定时器节点
+ Attribute Difference Agent 数值差异比较
+ Commander Agent 触发器代理，可以用于向其他节点发起指令控制，控制节点的执行和停止等
+ {{created_at}} 为自带抓取时间，Agent 设置中的特殊字符+，需要用反义符\\。


## 安装

### 手动部署
```shell
# 安装依赖包
apt-get update -y
apt-get upgrade -y
apt-get install sudo -y


# Install vim and set as default editor
sudo apt-get install -y vim
sudo update-alternatives --set editor /usr/bin/vim.basic


# Install the required packages
sudo apt-get install -y runit build-essential git zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate pkg-config cmake nodejs graphviz jq shared-mime-info


# Ubuntu 18.04 Bionic
sudo apt-get install -y runit-systemd


# Download Ruby and compile it:
mkdir /tmp/ruby && cd /tmp/ruby
curl -L --progress-bar https://cache.ruby-lang.org/pub/ruby/3.2/ruby-3.2.4.tar.xz | tar xJ
cd ruby-3.2.4
./configure --disable-install-rdoc
make -j`nproc`
sudo make install

sudo gem update --system --no-document
sudo gem install foreman --no-document


# Create a user for Huginn:
sudo adduser --disabled-login --gecos 'Huginn' huginn

```


### mysql 安装

```bash
# Install the database packages
sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev
# 报错的安装mysql分支，安装mariadb之后依旧是使用mysql命令，它是mysql的分支而已
lsb_release -a
apt install mariadb-server mariadb-client


# 逐步设置数据库 root 密码
sudo mysql_secure_installation

# 用上方设置的密码登陆数据库
mysql -u root -p
```

#### 配置
⚠️逐行输入代码到数据库命令行 `mysql>`，需将 `$password` 替换为你要设置的密码

```CREATE USER 'huginn'@'localhost' IDENTIFIED BY '$password';```
```SET default_storage_engine=INNODB;```
```GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, LOCK TABLES ON `huginn_production`.* TO 'huginn'@'localhost';```
```FLUSH PRIVILEGES;```
```\q```


数据库设置好后，拉取 huginn 主体程序，此段命令可以整段复制到 ssh。
```bash
# We'll install Huginn into the home directory of the user "huginn"
cd /home/huginn

# Clone Huginn repository
sudo -u huginn -H git clone https://github.com/huginn/huginn.git -b master huginn

# 如果有ruby 3.2等问题，可指定其他分支
# sudo -u huginn -H git clone https://github.com/huginn/huginn.git -b latest_rubygems huginn

# Go to Huginn installation folder
cd /home/huginn/huginn

# Copy the example Huginn config
sudo -u huginn -H cp .env.example .env

# Create the log/, tmp/pids/ and tmp/sockets/ directories
sudo -u huginn mkdir -p log tmp/pids tmp/sockets

# Make sure Huginn can write to the log/ and tmp/ directories
sudo chown -R huginn log/ tmp/
sudo chmod -R u+rwX,go-w log/ tmp/

# Make sure permissions are set correctly
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo -u huginn -H chmod o-rwx .env

# Copy the example Unicorn config
sudo -u huginn -H cp config/unicorn.rb.example config/unicorn.rb
```


sudo -u huginn -H editor .env 设置 huginn 环境依赖，更多选项查看 .env 设置案例。编辑器为上面安装的 vim，i 在光标所在的位置插入，esc 退出编辑，:wq 保存并退出。
```bash
DATABASE_ADAPTER=mysql2
#DATABASE_ENCODING=utf8   # 修改点
DATABASE_RECONNECT=true
DATABASE_NAME=huginn_production  # 修改点
DATABASE_POOL=20
DATABASE_USERNAME=huginn   # 修改点
DATABASE_PASSWORD='$password' # 修改点，换为你自己的密码
#DATABASE_HOST=your-domain-here.com
#DATABASE_PORT=3306
#DATABASE_SOCKET=/tmp/mysql.sock

# MySQL only: If you are running a MySQL server >=5.5.3, you should
# set DATABASE_ENCODING to utf8mb4 instead of utf8 so that the
# database can hold 4-byte UTF-8 characters like emoji.
DATABASE_ENCODING=utf8mb4  #修改点

...
RAILS_ENV=production  # 修改点

USE_GRAPHVIZ_DOT=dot # 取消注释，启用 GRAPHVIZ 来生成 diagram

TIMEZONE="Beijing" # bundle exec rake time:zones:local，时区需按指定格式填写，否则会报错 runsv not running

DEFAULT_HTTP_USER_AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36" # 浏览器访问

# 邮件发送设置
SMTP_DOMAIN=newzone.top
SMTP_USER_NAME=benson@newzone.top
SMTP_PASSWORD=somepassword
SMTP_SERVER=smtp.feishu.cn
SMTP_PORT=465
SMTP_AUTHENTICATION=plain
SMTP_ENABLE_STARTTLS_AUTO=true
SMTP_SSL=true
SEND_EMAIL_IN_DEVELOPMENT=true

# Maximum runtime of background jobs in minutes
# 默认为2分钟，不过如果你数据较多，有可能需要调整此项
DELAYED_JOB_MAX_RUNTIME=10
```

Install Gems 前用子账户重新设置运行目录权限 sudo chown -R huginn:huginn /home/huginn，防止报错 Your user account isn't allowed to install to the system RubyGems。
```bash
# 注意看黄字警告
gem install bundler
# Docker 环境中，时区容易丢失(6-70)
apt-get install tzdata
# Install Gems
sudo -u huginn -H bundle config set deployment 'true'
sudo -u huginn -H bundle config set without 'development test'
sudo -u huginn -H bundle install
# 备用 Gems 修复命令
# bundle update
# gem update bundler
# vim /home/huginn/huginn/Gemfile

# Initialize Database
# Create the database
sudo -u huginn -H bundle exec rake db:create RAILS_ENV=production

# Migrate to the latest version
sudo -u huginn -H bundle exec rake db:migrate RAILS_ENV=production

# ⚠️设置登陆账户密码，Create admin user and example agents using the default admin/password login
sudo -u huginn -H bundle exec rake db:seed RAILS_ENV=production SEED_USERNAME=admin SEED_PASSWORD=password

# Compile Assets
sudo -u huginn -H bundle exec rake assets:precompile RAILS_ENV=production
```


sudo -u huginn -H editor Procfile 修改 huginn 设置。如果需多现成运行，可移除 Multiple DelayedJob workers 部分的注释。
```bash
# 在下两行前，添加符号「#」
#web: bundle exec rails server -p ${PORT-3000} -b ${IP-0.0.0.0}
#jobs: bundle exec rails runner bin/threaded.rb

# 删除以下下两行前的符号「#」
web: bundle exec unicorn -c config/unicorn.rb
jobs: bundle exec rails runner bin/threaded.rb
```

'sv stop huginn-web-1' exited with a non-zero return value: fail: huginn-web-1: runsv not running 的报错，使用 foreman export runit -a huginn -l /home/huginn/huginn/log /etc/service 和 chown -R huginn:huginn /etc/service/huginn*。[2] [3] 如果是重启 Huginn 时出现此报错，则检查 sudo -u huginn -H editor .env 设置。

```bash
# 切换到
cd /home/huginn/huginn
# 设置
git config --global --add safe.directory /home/huginn/huginn
# 设置开机启动
sudo runsvdir /etc/service &
sudo bundle exec rake production:export

# Setup Logrotate
sudo cp deployment/logrotate/huginn /etc/logrotate.d/huginn

# Ensure Your Huginn Instance Is Running
sudo bundle exec rake production:status
```

Nginx 站点设置：
```bash
sudo apt-get install -y nginx

# Site Configuration
sudo cp deployment/nginx/huginn /etc/nginx/sites-available/huginn
sudo ln -s /etc/nginx/sites-available/huginn /etc/nginx/sites-enabled/huginn

# Change YOUR_SERVER_FQDN to the fully-qualified domain name of your host serving Huginn.
sudo editor /etc/nginx/sites-available/huginn

# 不需要 https，则改为下方配置
server {
  listen 80; # 监听的端⼝
  server_name localhost home.newzone.top; # 域名或ip，这里启用了两个地址，用空格分开

# 测试设置是否正确
sudo nginx -t

# 移除默认网站设置，只有当服务器/容器只存在 Huginn 网站方执行下行命令
sudo rm /etc/nginx/sites-enabled/default
```
以上完成了 Huginn 的所有部署，执行 `sudo service nginx restart` 即可访问网站。