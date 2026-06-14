# Introduction
---
* 在 ESP32 上构建 Web 服务器来响应来自客户端的 HTTP GET 请求。
---
* 使用 ESP-IDF 框架在 ESP32 上实现 Web 服务器。
* ESP32 充当 AP，其他工作站设备可以连接到它。
* 在 ESP32 上配置一个 Web 服务器，以便站点可以发出 http 请求以从中获取信息。
## 任务
* 将esp32 配置为接入点（AP）
* 配置esp32 http 服务器
# Configure esp32 as AP
* 参考[[softap]]
* 设置esp32的IP地址为192.168.1.1
# Configure esp32 HTTP Server
## Create a http server
* 分配资源并创建一个http服务器
* 服务器创建后，返回一个`httpd_handle_t`类型的句柄
```c++
httpd_config_t config=HTTPD_DEFAULT_CONFIG();
httpd_handle_t server=NULL;
if (httpd_start(&server, &config) == ESP_OK) {
	// Do something 
}
```
## Register URL handlers
* handle_是 httpd_handle_t 类型的变量，它引用您在上一步中创建的服务器。
* uri_handler_是指向 httpd_uri_t 类型的常量结构的指针。该结构包含 uri 的定义以及当客户端访问该 uri 时执行哪个回调函数。
```c++
httpd_uri_t test_uri={
            .uri="/test",
            .method=HTTP_GET,
            .handler=test_handler,
            .user_ctx=NULL
        };
        httpd_register_uri_handler(server, &test_uri);
```
## Implement URL callback
* 该响应返回一个包含 html h1 标记和 Hello World 文本的字符串。
```c++

esp_err_t test_handler(httpd_req_t *req)
{
    const char resp[]="<h1>Hello World</h1>";
    httpd_resp_send(req,resp,HTTPD_RESP_USE_STRLEN);
    return ESP_OK;
}
```

# Test the web server
* 连接该WiFi
* 打开浏览器并访问地址192.168.1.1/test。您应该在浏览器上看到 Hello World 消息。
* 地址应切换为你的esp32的IP地址
* 在连接成功后，给你的设备分配的IP应该是192.168.1.x
# Weapping Up