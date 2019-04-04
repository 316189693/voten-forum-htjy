Voten是一款基于laravel的php项目，类似于博客的系统,是一个开源项目.我们的unis-forum在voten的
基础上做二次开发.

### 代码：<br/>
  1. unis-forum: 
     > https://bitbucket.org/logisticsteam-dev/unis-forum/src/master/ 
  2. docker-file: 
     > https://bitbucket.org/logisticsteam-dev/plumber/src/release/data/dev_unis_forum/


### Jenkins: <br/>
  * url:
       > https://build.logisticsteam.com/jenkins/
  * account：
       > unisforum/unis4forum

### 服务器：<br/>
   * 域名： <br/>
        >forum-dev.unisco.com 
   * 本地登录服务器：<br/>
        > 用户是ubuntu，使用key文件wisereader.pem登录, key文件联系管理员获取，使用如下命令登录：
       
       ` ssh -i wisereader.pem ubuntu@forum-dev.unisco.com`

### 部署须知：
 * **配置分为两部分：** <br/>
  > 1. 代码端的env参数设置, <br/>
  > 2. docker file的参数设置. 
 * **docker container ip：** <br/>
   > docker container之间通过ip进行通讯, ip的产生和启动顺序有关。
  请按如下列出的顺序依次启动,并查看ip是否和下面的一致,否则就得修改对应的配置文件: 
   1. mysql：172.17.0.2
   2. redis：172.17.0.3
   3. elasticsearch：172.17.0.5
   4. laravel-echo-server：172.17.0.6
   5. php-fpm：172.17.0.8
   6. nginx：172.17.0.7 

* **查看IP:** 
   > `docker inspect 014ec1af3b14 |grep 172`

* **ip关联配置：**
 > 1. unis-forum的参数文件.env:
   >>>>  DB_HOST=172.17.0.2 (mysql的ip) <br/>
   >>  REDIS_HOST=172.17.0.3 <br/>
   >>  ELASTICSEARCH_HOST=http://172.17.0.5:9200 <br/>
 > 2. docker 文件：
   >> * data/dev_unis_forum/build/image_template/forum-laravel-echo-server/laravel-echo-server.json：
   >>> 1. "authHost": "52.12.95.183" （部署主机的ip，forum-dev.unisco.com=52.12.95.183）
   >>> 2. "redis": {
   			"port": "6379",
   			"host": "172.17.0.3",（redis ip）
   			"password": "SkIwalaNgRoN"
   }
   >>> 3. "host": "172.17.0.6" （laravel-echo-server ip）
 
 >> * data/dev_unis_forum/build/image_template/forum-nginx/conf.d/forumweb/unis-forum.conf
   >>> 1. location /socket.io {
         proxy_pass http://172.17.0.6:6001; （laravel echo server ip）#could be localhost if Echo and NginX are on the same box
         proxy_http_version 1.1;
         proxy_set_header Upgrade $http_upgrade;
         proxy_set_header Connection "Upgrade";
     }
     
>> * data/dev_unis_forum/build/image_template/forum-nginx/conf.d/app-upstream.conf:
  >>>   1. server 172.17.0.8:9000;(php-fpm ip)
 
 * Plumber:
 > 如果docker file,在执行自己创建的jenkins之前,先执行Plumber jenkins job来更新服务器上的文件.
 
### 如何部署？
 1. 建立jenkins
    创建了七个jenkins,一个代码,6个需要的软件.
 2. 按照【部署须知】顺序启动jenkins job
 3. 校验每个job是否启动正常,ip是否匹配【部署须知】的内容：
    docker ps; // 找到要校验的job
    docker logs [container id]；//
 4. 修改mysql
   进入mysql container：
   docker exec -it [container id] bash
   链接数据库：mysql -u root -p
   输入密码：root
   执行命令：
    update mysql.user set host = '%' where user='root'; 
 5. 给目录权限：<br/>
   sudo su - <br/>
  cd /var/lib/jenkins/workspace/dev-forum-workspace <br/>
  mkdir vendor <br/>
  chmod -R 777 vendor <br/>
  chown -R jenkins vendor <br/>
  chgrp -R jenkins vendor <br/>
  cd /var/lib/jenkins/workspace/dev-forum/storage <br/>
  chmod -R 777 logs <br/>
  cd /var/lib/jenkins/workspace/dev-forum/bootstrap <br/>
  cd /var/lib/jenkins/workspace/dev-forum/ <br/>
  cp .env-forum .env <br/>
  chmod -R 777 .env <br/>
  chmod -R 777 cache <br/>
  chmod -R 777 package.json <br/>
  mkdir node_modules <br/>
  chmod -R 777 node_modules <br/>
  chown -R jenkins node_modules
  chgrp -R jenkins node_modules <br/> 
  chmod -R 777 /var/lib/jenkins/workspace/dev-forum <br/>
 6. 进入php-fpm container,执行下面的命令：
   composer install <br/>
   php artisan key:generate <br/>
   php artisan passport:install <br/>
   php artisan es:init <br/> 
   npm install <br/>
   npm run production <br/>
   php artisan db:seed --class=AdminUserSeeder <br/>
   
   
   
   
   ### Create admin user
   
   To create an admin user, run the below command from the root of the project
   
   ```
   php artisan db:seed --class=AdminUserSeeder
   ```
   
   The login details for the admin user is `admin` and `qwer1234`.
   
   After running the seeder, be sure to clear your redis cache, you should now be able to navigate to `/backend`