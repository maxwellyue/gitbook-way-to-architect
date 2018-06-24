# 前端请求超时

## Ajax请求超时设置

使用jQuery来进行Ajax请求时，可以在请求时带上timeout参数设置超时时间。

```javascript
$.ajax({
    url:"http://xxx.xx.com:8080/test",
    dataType:"json",
    timeout:2000,
    success:function(result,status,xhr) {
       //success
    },
    error: function(result,status,xhr){
        if(status== 'timeout') {
            //timeout
        }
    }
});
```

当进行跨域JSONP请求时，使用jQuery 1.4.x版本时，IE9、Chrome 52、Firefox 49测试 JSONP时，请求在超时后不能被取消，即使客户端超时了，该脚本也将一直运行；使用jQuery1.5.2时超时是起作用了，但是，发出去的请求是没有取消的（请求还处于执行状态）。

## axios请求超时设置

> 默认为0，表示不限制，单位均为毫秒

全局设置：

```text
axios.defaults.timeout = 6000;
```

实例配置：

```javascript
const instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

请求配置：

```javascript
axios({
  method:'get',
  url:'http://bit.ly/2mTM3nY',
  timeout:1000,
  responseType:'stream'
})
  .then(function(response) {
  response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
});
```



## 参考

[axios请求超时，设置重新请求的完美解决方法](https://juejin.im/post/5abe0f94518825558a06bcd9)

[axios](https://github.com/axios)/[**axios**](https://github.com/axios/axios)

《亿级流量网站架构核心技术》：超时与重试机制



