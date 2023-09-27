## 1. ngx_http_proxy_module模块中配置proxy_pass
http模块中配置proxy_pass指令用于转发http请求，配置在nginx.conf中的http模块下   
若prox_pass为相对路径不带斜线/，http://192.168.16.40:5000/api/getData，则后端request_uri为/api/getData  
```nginx
server {
  listen 5000;
  location /api/ {
    proxy_pass http://192.168.16.40:5999;
  }
}
```
prox_pass为绝对路径带/，访问http://192.168.16.40:5000/api/getData，则后端request_uri为/getData   
```
server {
  listen 5000;
  location /api/ {
    proxy_pass http://192.168.16.40:5999/;
  }
}
```



## 2. ngx_stream_proxy_module模块中配置proxy_pass
**ngx_stream_proxy_module模块用于创建tcp或udp请求用于连接其他服务，比如redis,mysql；配合proxy_pass指令用于转发请求，该指令只能在server段使用使用, 只需要提供域名或ip地址和端口，文档地址：[https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html)**  
例如将浏览器的http请求地址通过tcp转到另一个http服务器地址：   
nginx.conf中需要有stream模块（与http模块同级），如果没有则手动添加模块
```nginx
stream {
  #打印日志格式
  log_format basic '$remote_addr [$time_local] ' '$protocol $status $bytes_sent $bytes_received ' '$session_time';
  #设置日志文件
  access_log logs/stream-access.log basic buffer=32k;
  error_log logs/stream-error.log;
  #stream文件一般单独放在一个文件夹，这里用include引入
  include vhosts/*.stream;
}
```
在vhost中新建ngtest.stream，设置proxys_pass设置tcp转发目标服务器 
```nginx
server {
  listen 5000;
  #将目标服务器的地址设置为以下地址
  proxy_pass 192.168.16.40:5999;
}
```
5999端口http配置参考
```nginx
#目标地址
server {
  listen 5999;
  index index.html;
  root D:/project/ng/proxy_pass/5999;
}
```
重新加载nginx配置  
访问http://192.168.16.40:5000，代理到http://192.168.16.40:5999  
如果访问的是http://192.168.16.40:5000/test.html，则代理到http://192.168.16.40:5999/test.html  
***也由此可见，stream模块的proxy_pass指令不会改变源请求协议，只做ip和端口转发***