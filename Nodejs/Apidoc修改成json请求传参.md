# Apidoc修改成json请求传参

- 进入全局目录

```
cd /usr/local/lib/node_modules/apidoc
```

- 修改`template/utils/send_sample_request.js`文件

修改`sendSampleRequest`方法

1. 修改`Optional header`对象代码如下

```
// Optional header
var header = {};
header["Content-Type"]="application/json;charset=UTF-8"   // 增加了这一行
$root.find(".sample-request-header:checked").each(function(i, element) {
    var group = $(element).data("sample-request-header-group-id");
    $root.find("[data-sample-request-header-group=\"" + group + "\"]").each(function(i, element) {
    var key = $(element).data("sample-request-header-name");
    var value = element.value;
    if ( ! element.optional && element.defaultValue !== '') {
        value = element.defaultValue;
    }
    header[key] = value;
    });
});
```

2. 修改`ajaxRequest`对象中的data参数

```
// send AJAX request, catch success or error callback
var ajaxRequest = {
    url        : url,
    headers    : header,
    data       : JSON.stringify(param),  // 从param改为JSON.stringify(param)
    type       : type.toUpperCase(),
    success    : displaySuccess,
    error      : displayError
};
```


