个人项目 pwd manager 的前端页面，选用了 amis 前端低代码框架。

- [amis Github页面](https://github.com/baidu/amis)
- [amis 文档](https://aisuda.bce.baidu.com/amis/zh-CN/docs/index)

使用的是 amis sdk，版本 3.4.3

# JWT 身份验证

引入 axios cdn

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

## login.html

在amis.embed的第四个参数中这样写

```js
fetcher: (url, method, data, config) => {
// 发起请求使用 axios
    return axios(url, method, data, config)
    .then(response => {
        if (response.data.status === 0) {
            // 提取JWT Token，假设Token存储在响应的data字段中
            const token = response.data.data.token;

            if (token) {
                // 将Token存储在localStorage中
                localStorage.setItem('token', token);
                console.log(token);
            }
        }
        return response; // 返回响应对象
    })
    .catch(error => {
        // 处理请求错误
        console.error('请求错误', error);
        throw error;
});
```

## index.html

在每个请求中都加入

```json
"headers": {
	"token": "${ls:token}"
}
```

例如：

```json
"api": {
    "url": "/api/account/page",
    "method": "get",
    "headers": {
        "token": "${ls:token}"
    }
}
```

在amis.embed的第四个参数中这样写，我这里设置的 jwt 验证不通过后端返回401

```js
fetcher: (url, method, data, config) => {
    const token = localStorage.getItem("token");
    // 如果token不存在，跳转index.html
    if (!token) {
        window.location.href = '/login.html';
    }

    return axios(url, method, data, config)
        .catch(error => {
            if (error.response && error.response.status === 401)
                window.location.href = '/login.html';
        });
}
```

