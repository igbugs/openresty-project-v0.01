![Markdown](https://github.com/Tinywan/lua_project_v0.01/blob/master/public/images/tinywan_title.png)
------
##  Project structure
```javascript
.
├── application                      -- 业务代码
│   ├── get_redis_iresty.lua
│   ├── ip_location.lua
│   ├── lua_websocket_server.lua
│   ├── mysql
│   ├── mysql.lua
│   ├── redis
│   ├── swoole
│   ├── test.lua
│   ├── websocket_server_test.lua
│   └── zabbix
├── bin                             -- 脚本文件
│   ├── start.sh
│   └── stop.sh
├── conf                            -- 配置文件
│   ├── domains
│   ├── nginx.conf
│   └── nginx.conf.default
├── logs                            -- 日志文件 
│   ├── lua2_error.log
│   └── lua_error2.log
├── lualib                          -- 公共Lua库
│   ├── cjson.so
│   ├── ngx
│   ├── rds
│   ├── redis
│   ├── resty
│   └── vendor                      -- 第三方Lua库
├── public                          -- 公共静态文件
│   ├── data
│   ├── images
│   └── websocket
├── README.md                      
└── template                        -- 模板
```
## nginx.conf, the Nginx web server configuration

```lua
user  www www;
worker_processes  8;

pid        logs/nginx.pid;

events {
    use epoll;
    worker_connections  1024;
}

http {
    include       /opt/openresty/nginx/conf/mime.types;
    default_type  text/html;
    lua_package_path "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/lualib/?.lua;;";  #lua 模块
    lua_package_cpath "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/lualib/?.so;;";  #c模块
    include "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/conf/domains/*";
}
```
##  :date:
####    2017年05月07日 星期日 
+   创建项目、项目架构搭建、目录结构调整
+   添加新功能
    - [x] 简单的Redis数据库操作 
    - [x] 根据IP地址获取具体城市信息(@icowan)
    - [x] lua-resty-shell库的使用
####    2017年05月08日 星期一
+   添加功能：lua-resty-websocket 官方测试案例
+   添加功能：WebSocket系统负载统计
####    2017年05月10日 星期三
+   [Redis授权登录使用短连接(5000)和长连接(500W) 使用连接池AB压力测试结果](http://www.cnblogs.com/tinywan/p/6838630.html)        
####    2017年05月11日 星期四
+   Mysql 数据库添加
    +   错误提示：`bad result: Data too long for column 'name' at row 1: 1406: 22001.`,插入字段超过表定义大小
####    2017年05月14日 星期日
+   Swoole WebSocket 测试
    +   开启服务：`php /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/application/swoole/websocket-server.php`
    +   客户端测试：`http://127.0.0.1/clinet.html`
####    2017年05月15日 星期一
+   添加Nginx 启动脚本：`start.sh`
    +   开启Nginx：`/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/bin/start.sh`
    +   如果已经开始Nginx，则执行`start.sh `可以重新加载配置文件
+   添加Nginx 停止脚本：`stop.sh`
    +   停止Nginx：`/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/bin/stop.sh`
+   脚本运行错误：
    +   提示错误：` /bin/bash^M: 解释器错误: 没有那个文件或目录`
    +   解决办法：`sed -i 's/\r$//' ../lua_project_v0.01/bin/start.sh`
    +   [解决 linux下编译make文件报错“/bin/bash^M: 坏的解释器：没有那个文件或目录” 问题](http://blog.csdn.net/liuqiyao_01/article/details/41542101#comments)
    
####    :fallen_leaf: 2017年05月17日 星期三
+   ngx_lua_waf 安装使用(ngx_lua_waf是一个基于lua-nginx-module(openresty)的web应用防火墙) 
    +   [ngx_lua_waf github 地址](https://github.com/loveshell/ngx_lua_waf)
    +   安装步骤
        1.  下载源码到某一目录：复制目录到`application`下面，同时重命名`waf`
        2.  命令：`cp -R ngx_lua_waf /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/application/waf`
        3.  移动源文件目录中的`config.lua`文件到`../lualib/vendor`下
        4.  修改waf 目录下的`init.lua`文件的第一行：`require 'config'`修改为`require 'vendor.config'`
        5.  在`domains`目录下,新建文件`waf.conf`,添加以下内容
            ```lua
                lua_shared_dict limit 10m;
                init_by_lua_file  "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/application/waf/init.lua";
                access_by_lua_file "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/application/waf/waf.lua";
                
                server {
                    listen       8082;
                    server_name  localhost;
                    index  index.php index.html index.htm;
                    access_log  /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/logs/waf_access.log;
                    error_log /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/logs/waf_error.log error;
                
                    set $web_root /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/template/waf;
                    root $web_root;
                
                    location / {
                        try_files $uri $uri/ /index.php?$args;
                    }
                
                    location ~ \.php$ {
                        fastcgi_pass   unix:/var/run/php7.0.9-fpm.sock;
                        fastcgi_index  index.php;
                        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                        include        fastcgi_params;
                    }
                }
            ```
        6.  重启Nginx即可 
           ```bash
            tinywan@tinywan:~$ /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/bin/start.sh 
            [ Nginx running ] 
            nginx: the configuration file /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/conf/nginx.conf syntax is ok
            nginx: configuration file /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/conf/nginx.conf test is successful
            [ Nginx has reload ]
           ```
        7.  可能会遇到的错误
            -   [1]`nginx: [emerg] open() "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/conf/fastcgi_params" failed`
                 >  解决：`cp fastcgi_params /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/conf/fastcgi_params`
            -   [2]`FastCGI sent in stderr: "Primary script unknown" while reading response header from upstream`
                 >  解决：查看`waf.conf`配置信息是否正确 

+   修改文件：`config.lua`
    ```lua
    black_fileExt={"php","jsp"}
    ipWhitelist={"127.0.0.1"}
    --ipBlocklist={"1.0.0.1"}
    ipBlocklist={"192.168.127.133"}
    ```  
+   测试一：`http://127.0.0.1:8082/waf.php?id=../etc/passwd&name=Tinywan`     
    -   ![waf_ip_blaklist3](https://github.com/Tinywan/lua_project_v0.01/blob/master/public/images/github/waf_ip_blaklist3.png)
+   测试二：`http://192.168.127.133:8082/waf.php?id=../etc/passwd&name=Tinywan`     
    -   ![waf_ip_blaklist2](https://github.com/Tinywan/lua_project_v0.01/blob/master/public/images/github/waf_ip_blaklist2.png)
+   :poop: 坑 :poop: 在提交代码的时候`waf`目录一直提交不了，提示:`modified: xxx(modified content, untracked content)`,
    原来在`waf`目录下有个.git 目录,删除.git目录,重新git add 就可以了   
####    :hibiscus: 2017年05月19日 星期五
*  第一版采用单模块设计
*  简单的MVC模式，目录、命名约定（Phalcon MVC）
    +   Model（模型）负责在数据库中存取数据 
    +   View（视图）是应用程序中处理数据显示的部分     
    +   Controller（控制器）通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据   
* 第一个简单的 mvc 模式的文件路径访问
1. 配置文件：`nginx_product.conf`  

    ```bash
    #upstream
    upstream item_http_upstream {
        server 192.168.1.1 max_fails=2 fail_timeout=30s weight=5;
        server 192.168.1.2 max_fails=2 fail_timeout=30s weight=5;
    }
    
    #缓存 共享字典配置
    lua_shared_dict item_local_shop_cache 600m;
    
    # server product
    server {
        listen       8082;
        server_name  127.0.0.1;
        charset gbk;
        index  index.html index.htm;
        access_log  /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/logs/product_access.log;
        error_log /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/logs/product_error.log error;
    
        #加载模板文件
        set $template_root /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/template;
    
        #url映射
        location ~* "^/product/(\d+)\.html$" {
            rewrite /product/(.*)    http://127.0.0.1:8082/$1 permanent;
        }
    
        # chapter
        location ~* "^/(\d{6,12})\.html$" {
            default_type text/html;
            lua_code_cache on;
            content_by_lua_file "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/application/controller/ProductController.lua";
        }
    }
    ```
2. 当我们访问页面：`http://127.0.0.1:8082/13669361192.html`  
   将交给`lua_project_v0.01/application/controller/ProductController.lua`处理  
    
    ```html
    curl -k http://127.0.0.1:8082/13669361192.html
    Hello ProductController.lua
    uri = /13669361192.html
    ```
3. 项目入口搞定 :hibiscus: :hibiscus: :hibiscus: :hibiscus:        
 
* :bouquet: lua-resty-template 的使用    

    ```lua
        #加载模板文件
        set $template_root "/mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/template/product";
      
        location /template_test {
           default_type 'text/html';
           content_by_lua '
                local template = require "resty.template"
                template.render("index.html", { message = "Hello, World!" })
           ';
        }
    ```
* :blossom: `resty.template`渲染模板,通过`ngx API`输出内容到指定的`html`页面  
    -  ![Markdown](https://github.com/Tinywan/lua_project_v0.01/blob/master/public/images/github/lua-resty-template.png)

      
#### 　:deciduous_tree: 2017年05月20日 星期六
+   :white_check_mark: helper类的封装
    + Example
      ```lua
      local helper = require 'vendor.helper'
      local array = helper.split('a,b,v,b',',')
      for key,value in ipairs(array)
      do
        ngx.say(key, value)
      end
      ```     
+   :white_check_mark: dkjson 库的加入 :clap: :clap: :clap:  
    +   [常用Lua开发库2-JSON库、编码转换、字符串处理](http://jinnianshilongnian.iteye.com/blog/2187643)
    +   Example 
        ```lua
        local dkjson = require 'vendor.dkjson'
        --lua对象到字符串
        local obj = {
            id = 110,
            name = "Tinywan",
            age = 24,
            is_male = false,
            hobby = {"film", "music", "read"}
        }
        local str = dkjson.encode(obj, {indent = true})
        ngx.say(str, "<br/>")
        ```
        
## 功能列表
####    简单的Redis数据库操作  
+   通过引入已经封装好的Redis类操作Redis数据
+   直接运行curl:`curl http://127.0.0.1/get_redis_iresty`
+   官方短连接和长连接测试已通过（本地环境）
+   官方短连接和长连接测试已通过（本地环境）
####    根据IP地址获取具体城市信息
+   根据IP地址，获取具体IP地址的城市详细信息
+   直接运行curl:`curl http://127.0.0.1/test_ip_location?ip='122.228.95.112'`
####    lua-resty-shell 库
+   执行shell命令
+   获取Linux的CPU信息：`curl http://127.0.0.1/shell_test`   
####    lua-resty-websocket 官方测试案例
+   服务端文件`lua_websocket_server.lua`,客户端接受文件`public/index_default.html`,JS代码`public/static/stats_default.js`
+   服务端配置location
    ```lua
       location /lua_websocket_server {
           default_type text/html;
           content_by_lua_file /mnt/hgfs/Linux-Share/Lua/lua_project_v0.01/application/lua_websocket_server.lua;
       }
    ```
+   客户端，通过js`console.log(e.data)` 打印接收到的数据，这类我们可以接受到服务端发送的消息
   ```lua
   -- 服务端发送文本
   bytes, err = wb:send_text("Hello world Tinywan")
   
   --客户端接受数据，通过浏览器Console 可以看到以下数据
   Hello world Tinywan
   stats_default.js:70 Blob {size: 17, type: ""}
   stats_default.js:65 disconnect
   stats_default.js:60 connect
   ```
####    websocket系统负载统计
+   基于lua-resty-websocket实现系统负载监控
+   使用第三方编写的websocket_shell.lua(原本叫shell.lua) 一个库文件
+   第三方库或者文件引入：`local shell = require 'vendor.websocket_shell'`
+   注意：在这里并没有用到`lua-resty-shell` 库，有时间整合到一起去 
+   效果预览图：
    ![websocket_shell](https://github.com/Tinywan/lua_project_v0.01/blob/master/public/images/github/WebSocket_shell.jpg)
