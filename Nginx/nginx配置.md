## nginx 配置



### server

每个域名对应配置一个 server



### proxy_pass

在nginx中配置proxy_pass代理转发时，如果在proxy_pass后面的url加/，表示绝对根路径；如果没有/，表示相对路径，把匹配的路径部分也给代理走。

假设下面四种情况分别用 http://192.168.1.1/proxy/test.html 进行访问。

第一种：
 location /proxy/ {
 proxy_pass http://127.0.0.1/;
 }
 代理到URL：http://127.0.0.1/test.html

第二种（相对于第一种，最后少一个 / ）
 location /proxy/ {
 proxy_pass http://127.0.0.1;
 }
 代理到URL：http://127.0.0.1/proxy/test.html

第三种：
 location /proxy/ {
 proxy_pass http://127.0.0.1/aaa/;
 }
 代理到URL：http://127.0.0.1/aaa/test.html

第四种（相对于第三种，最后少一个 / ）
 location /proxy/ {
 proxy_pass http://127.0.0.1/aaa;
 }
 代理到URL：http://127.0.0.1/aaatest.html



### 配置示例

// 前端路由

location /iot-data-open {
        proxy_pass https://172.29.1.55:9001/iot-data-open;
}

// http 路由

location /aiot/platform {
        proxy_pass http://172.29.1.54:8891;
        proxy_http_version 1.1;
        proxy_read_timeout 3600s;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_ignore_client_abort on;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
}

// ws 路由、协议升级

location /aiot/platform/ws/ {
        proxy_pass http://172.29.1.52:8925/ws/;
        proxy_http_version 1.1;
        proxy_read_timeout 3600s;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host:$server_port;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
