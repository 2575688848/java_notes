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

