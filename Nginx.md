## Nginx

### 五个优点

- 高并发，高性能
- 可扩展性好【模块化设计，其他三方插件生态圈丰富】
- 高可靠性【可以在服务器端持续不断的运行数年】
- 热部署【这个功能对于Nginx来说特别重要，热部署指可以通过不停止Nginx服务的情况下升级Nginx】
- BSD许可证【意味着我们可以将源码下载下来进行修改然后使用自己的版本】

### 四个主要组成部门

- Nginx二进制可执行文件：由各模块源码编译出一个文件
- Nginx.conf配置文件：控制Nginx行为
- acess.log访问日志： 记录每一条HTTP请求信息
- error.log 错误日志：定位问题

### 负载均衡的方式

- 默认

  ```java
  upstream test {
          server localhost:8080;
          server localhost:8081;
  }
  ```

- 权重

  ```java
  // 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
  upstream test {
          server localhost:8080 weight=9;
          server localhost:8081 weight=1;
  }
  ```

- ip_hash

  ```java
  // iphash的每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
  upstream test {
      ip_hash;
      server localhost:8080;
      server localhost:8081;
  }
  ```

- fair

  ```java
  // 按后端服务器的响应时间来分配请求，响应时间短的优先分配。
  upstream backend { 
          fair; 
          server localhost:8080;
          server localhost:8081;
  }
  ```

- url_hash【第三方】

  ```java
  //按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。 在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
  upstream backend { 
          hash $request_uri; 
          hash_method crc32; 
          server localhost:8080;
          server localhost:8081;
  }
  ```

## 