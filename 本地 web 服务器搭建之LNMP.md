# 0x01 任务

## 1. 环境搭建过程

**目标环境**

linux+nginx+php- fpm+mysql

### 1.1 安装 Linux 系统

Linux 系统使用 Ubuntu-18.04 版本，使用 VMware Workstation 安装其镜像，Ubuntu 镜像在其官网上下载。

![1564387874274](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564387874274.png)

安装完成后，进行系统和软件更新，在终端中输入：

```
sudo apt-get update & sudo apt-get upgrade
```

###  1.2 安装 Nginx 

在 Ubuntu 中安装 Nginx，只需要在终端输入以下命令即可：

```
sudo apt-get install nginx
```

在浏览器中浏览 nginx 默认网页，http://主机地址，说明安装成功

![1564392035766](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564392035766.png)

### 1.3 安装 Php-fpm 

在终端输入以下命令即可：

```
sudo apt-get intall php-fpm
```

查看服务是否运行：

```
systemctl status php7.2-fpm.service 
```

![1564392367696](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564392367696.png)

### 1.4 安装 Mysql

在终端输入以下命令安装 mysql 和 php 的 mysql 支持：

```
sudo apt-get install mysql-server mysql-client php-mysql
```

安装完成后，检查 mysql 服务器是否运行：

```
sudo systemctl mysql.service
```

![1564659362935](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564659362935.png)

## 2. 配置自己的网站

### 目标

搭建本地网站 www.jaylen.com，配置 Nginx 使其支持 php，本地网站显示一张图片，图片下方有个评论界面，可以输入评论，显示到网页中，评论会存到 mysql 数据库中。

### 2.1 域名映射

要在本地网络中，通过浏览器访问到 www.jaylen.com，需要修改主机的 hosts 文件，在 `/etc/hsots` 添加一行：

```
your-ip-add www.jaylen.com
```

这样就不需要 DNS 服务器解析该域名了（域名并没有注册！！！，DNS 服务器是没有记录的）。

### 2.2 配置 Nginx 

首先需要配置 Nginx，使其支持 php，在 `/etc/nginx/conf.d` 目录下添加自己网站服务器的配置，`jaylen.com.conf` ，内容如下：

```shell
 #  server configuration
 server {
     listen 80;
     listen [::]:80;
     server_name jaylen.com;
     root /var/www/jaylen.com; # 网站在主机的根目录
 
     # Add index.php to the list if you are using PHP
     index index.html index.htm index.nginx-debian.html index.php;
 
     location / {
         # First attempt to serve request as file, then
         # as directory, then fall back to displaying a 404.
         try_files $uri $uri/ =404;
     }
 
     # pass PHP scripts to FastCGI server
     location ~ \.php$ {
         include snippets/fastcgi-php.conf;
     
         # With php-fpm (or other unix sockets):
         fastcgi_pass unix:/run/php/php7.2-fpm.sock;
     }
 
 }
```

测试配置是否成功：

```shell
sudo nginx -t
```

成功的话，出现如下提示信息：

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

重新加载配置：

```shell
sudo nginx -s reload
```

### 2.3 创建网站主页

首先为网站创建一个根目录 `/var/www/jaylen.com` ，用于存储网站内容，该目录和 Nginx 配置文件中的 `root` 指令的内容一致。

```shell
sudo mkdir /var/www/jaylen.com
```

写个简单的 html 页面和 php进行测试，`index.html` ， 

```html
<html>
     <body>
         <head>
             <title>
                 jaylen's homepage
             </title>
         </head>
         <h1>Welcome to Jaylen's Homepage</h1>
         <p>Website is under construction, wait...</p>
     </bdoy>
 </html>
```

`index.php`

```php+HTML
<html>
     <body>
         <head>
             <title>
                 jaylen's homepage
             </title>
         </head>
         <h1>Welcome to Jaylen's Homepage</h1>
         <p>Website is under construction, wait...</p>
         <?php
             echo phpinfo();
         ?>
     </bdoy>
 </html>
```

通过浏览器浏览网站 www.jaylen.com

![1564820064946](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564820064946.png)

浏览 www.jaylen.com/index.php

![1564820032466](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564820032466.png)

测试完成，没有问题。

进一步完善网页内容，使其显示一张图片，图片下方带有评论输入框，可以提交评论，提交的评论存到主机的 mysql 数据库中，而后显示在评论区，最后完成的大致如下图，简陋的不行。

![1564840770425](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564840770425.png)

在 mysql 中创建一个新用户，并授权 INSERT、SELECT 权限，评论内容需要存到数据库，并读取数据库内容。

```mysql
mysql>
CREATE USER 'user101'@'localhost'
  IDENTIFIED BY 'webdev101@Webdev102';
GRANT INSERT,SELECT
  ON *.*
  TO 'user101'@'localhost'
  WITH GRANT OPTION;
```

刷新授权表，退出，

```mysql
mysql> FLUSH PRIVILEGE;
mysql> quit;
```

以新创建的用户重新登陆

```bash
mysql -u user101 -p
```

创建一个数据库

```mysql
mysql> create database webdb101
```

切换到该数据库

```mysql
mysql> use webdb101
```

创建一个表格，用于存放评论内容

```
mysql> CREATE TABLE comments (
id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
comment VARCHAR(200) NOT NULL
);
```

插入一条数据

```
mysql> INSERT INTO comments(comment) VALUES ('test');
```

最终的 `index.php` 文件内容如下：

```php+HTML
<!DOCTYPE HTML>
 <html>
 <head>
 <style>
 .error {color: #FF0000;}
 image {
     width: 100%;
     height: auto;
 }
 </style>
 </head>
 <body>
 
 <?php
 // define variables and set to empty values
 
 include('connect-mysql.php');
 
 $comment = "";
 $commentErr = "";
 
 if ($_SERVER["REQUEST_METHOD"] == "POST") {
   if (empty($_POST["comment"])) {
     $commentErr = "You must write someting...";
   } else {
     $comment = test_input($_POST["comment"]);
     $sql = "INSERT INTO comments (comment) VALUES ('".$comment."')";
     if (mysqli_query($dbcon, $sql)) {
         $commentErr = "Submit comment successfully!";
     }
     else {
         $commentErr = "Error:" . $sql . "<br>" . mysqli_error($dbcon);
     }
   }
 }
 
 function test_input($data) {
   $data = trim($data);
   $data = stripslashes($data);
   $data = htmlspecialchars($data);
   return $data;
 }
 ?>
 
 <h1>Jaylen's HomePage</h1>
 <img src='image/homepage.jpeg' alt='homepage icon' width='500px' height='400px'> 
 <form method="post" action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]);?>">  
   <br><br>
   Comment: <textarea name="comment" rows="5" cols="40"><?php echo $comment;?></textarea>
   <span class='error'>*<?php echo $commentErr; ?></span>
   <br><br>
   <input type="submit" name="submit" value="Submit">  
 </form>
 
 <?php
 
 $sqlget = "SELECT * FROM comments";
 $sqldata = mysqli_query($dbcon, $sqlget) or die("Fail to connect to database!" .  mysqli_error($dbcon));
 echo "<table>";
 echo "<tr><th>Comments</th></tr>";
 
 while($row = mysqli_fetch_array($sqldata, MYSQLI_ASSOC)) {
     echo "<tr><td>";
     echo $row['comment'];
     echo "</td></tr>";
 }
 
 echo "</table>";
 ?>
 
 </body>
 </html>

```

连接数据库的操作，单独放到 `connec-mysql.php` 文件中。

在网页中测试下提交评论，

![1564841954745](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564841954745.png)

### 2.5 问题

记录搭建自己本地网络过程中遇到的问题。

1. 配置 Nginx 使其支持 php 时，访问 http://localhsot/index.php，出现 `502 Bad Gateway` 错误，查看 Nginx 错误日志 `/var/log/nginx/error.log` ，找到错误记录如下：

   ```
   2019/08/02 03:11:53 [crit] 4293#4293: *50 connect() to unix:/var/run/php/php7.0-fpm.sock failed (2: No such file or directory) while connecting to upstream, client: 192.168.47.129, server: _, request: "GET /index.php HTTP/1.1", upstream: "fastcgi://unix:/var/run/php/php7.0-fpm.sock:", host: "192.168.47.129"
   ```

   原因是找不到 `unix:/var/run/php/php7.0-fpm.sock`文件，而我安装的版本是 7.2 的，因此应该是配置错误了，在系统中查找 `php7.0-fpm.sock`  发现实际路径为 `/run/php/php7.0-fpm.sock` 修改 `/etc/nginx/sites-avalable/default` 文件如下:

   ```shell
           # pass PHP scripts to FastCGI server
   
           location ~ \.php$ {
                   include snippets/fastcgi-php.conf;
   
                   # With php-fpm (or other unix sockets):
                   fastcgi_pass unix:/run/php/php7.2-fpm.sock;
                   # With php-cgi (or other tcp sockets):
                   #fastcgi_pass 127.0.0.1:9000;
           }
   ```

2. 在  `index.php` 文件中对数据库进行插入操作时，提示错误：

   ```
   Unknown column '' in 'field list'
   ```

   原因时插入的数据是 `VARCHAR` 类型，需要额外添加双引号，我原来的 sql 语句如下：

   ```php
   $sql = "INSERT INTO comments (comment) VALUES ($comment)";
   ```

   这样实际导致 `$comment` 变量展开后是一个标识符，而不是字符串，需要在外面添加额外的双引号，具体处理如下，注意在单引号中 `$` 不会被识别成特殊字符，因此不会展开变量，而双引号中不能直接包含双引号，所以就成了下面的结果。

   ```php
   $sql = "INSERT INTO comments (comment) VALUES ('".$comment."')";
   ```

   

## 3.本地服务器加固

### 3.1 加固 Linux 系统

待补充。

### 3.2 加固 Nginx

**关闭 Server Token**

Nginx 默认开启 Server Token（显示版本号），这样使得 Nginx 的版本号很容易被获取，如下图为连接域名不存在资源时的返回页面，可以看到 Nginx 的版本号

![1564802848522](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564802848522.png)

在 `/etc/nginx/nginx.conf` 中 http 块中添加（去掉注释即可）：

```shell
server_tokens off	
```

关闭后，访问域名下不存在的资源，返回页面中没有了 Nginx 的版本号 信息。

![1564815315221](C:\Users\jaylen_west\AppData\Roaming\Typora\typora-user-images\1564815315221.png)

### 3.2 加固 mysql

运行 ` mysql_secure_installation` 工具（安装 mysql 后自带的 shell 脚本），进行 mysql 的安全检查，

根据提示，设置密码为最高级别，并为 root 用户设置密码，最后同意以下选项：

- Remove anonymous users? -- 删除匿名用户
- Disallow root login remotely? -- 禁止远程使用 root 用户登陆
- Remove test database and access to it? -- 删除测试数据库和其访问权限
- Reload privilege tables now? -- 重载授权表

### 3.3 加固 PHP

Nginx 对于 PHP 支持的配置文件中常常会使用如下形式的配置，该配置使得 PHP 解释器接受所有以 `.php`结尾的 URI，这样一来就会存在很大的风险，存在任意代码执行漏洞，具体解释见参考资料 [6]。

修改 `/etc/php/7.2/fpm/php.ini`文件（不同系统该文件位置略有不同），设置`cgi.fix_pathinfo=0`，可以禁止 PHP 解释器查找文件系统中不存在的文件，使用sed 命令完成文件内容的修改：

```bash
sudo sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' /etc/php/7.2/fpm/php.ini
```

```
sh ~/.vim_runtime/install_awesome_vimrc.sh
sh ~/.vim_runtime/install_awesome_parameterized.sh /opt/vim_runtime --all
```

# 0x02 参考资料

[1] [Install a LEMP Stack on Ubuntu 18.04](https://www.linode.com/docs/web-servers/lemp/how-to-install-a-lemp-server-on-ubuntu-18-04)

[2] [Serve PHP with PHP-FPM and NGINX](https://www.linode.com/docs/web-servers/nginx/serve-php-php-fpm-and-nginx/ )

[3] [Nginx vs Apache](https://www.keycdn.com/support/nginx-vs-apache )

[4] [Setting up an Nginx Reverse Proxy](https://www.keycdn.com/support/nginx-reverse-proxy )

[5] [Getting  Started with NGINX](https://www.linode.com/docs/web-servers/nginx/nginx-installation-and-basic-setup/#configuration-recap)

[6] [Passing Uncontrolled Requests to PHP](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/?highlight=pitfalls#passing-uncontrolled-requests-to-php )

[7] [Unknown column &#39;&#39; in &#39;field list&#39;解决方案](https://blog.csdn.net/qq_15936309/article/details/51859761 )

