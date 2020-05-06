## 生命周期

1. 初始化

   调用init()方法来进行初始化

   ​		init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用

   ​		Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

2. 处理请求

   调用service()方法来处理客户端请求

   ​		执行实际任务的主要方法，Servlet容器（即web服务器）调用service（）方法来处理来自客户端的请求，并把格式化的响应回写给客户端

   ​		每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

   ​		不用对 service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

3. 终止

   调用destroy()方法终止

   ​		destroy()方法只会被调用一次，在Servlet生命周期结束时被调用，方法可以让您的Servlet关闭数据库连接，停止后台线程，把Cookie列表或者点击计数器写入到磁盘，

   ​		在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。

4. 回收

   由JVM的垃圾回收器进行垃圾回收