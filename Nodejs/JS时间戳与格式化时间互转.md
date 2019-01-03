# JS时间戳与格式化时间互转

## Javascript 获取当前时间戳（毫秒级别）

```
// 方法一(精确到秒)
var timestamp1 = Date.parse( new Date());

// 方法二(精确到毫秒)
var timestamp2 = ( new Date()).valueOf();

// 方法三(精确到毫秒)
var timestamp3 = new Date().getTime();
```

## 时间戳转成格式化时间
```
const dateFormat = function(dateStr, fmt) {
  let date = new Date(dateStr)
  var o = {
    "Y+": date.getFullYear(),
    "M+": date.getMonth() + 1,                 //月份
    "D+": date.getDate(),                    //日
    "h+": date.getHours(),                   //小时
    "m+": date.getMinutes(),                 //分
    "s+": date.getSeconds(),                 //秒
    "q+": Math.floor((date.getMonth() + 3) / 3), //季度
    "S+": date.getMilliseconds()             //毫秒
  };
  for (var k in o) {
    if (new RegExp("(" + k + ")").test(fmt)) {
      if (k == "Y+") {
        fmt = fmt.replace(RegExp.$1, ("" + o[k]).substr(4 - RegExp.$1.length));
      }
      else if (k == "S+") {
        var lens = RegExp.$1.length;
        lens = lens == 1 ? 3 : lens;
        fmt = fmt.replace(RegExp.$1, ("00" + o[k]).substr(("" + o[k]).length - 1, lens));
      }
      else {
        fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
      }
    }
  }
  return fmt;
}
```

## 格式化时间转成时间戳

```
"2016-08-03 00:00:00" ——> 1470153600000
```

```
let dateF = "2016-08-03 00:00:00"
let date = new Date(dateStr)
let dateS = date.valueOf()
console.log(dateS)  // 1470153600000
```


## 参考文章

- [JavaScript获取时间戳与时间戳转化](https://segmentfault.com/a/1190000006160703)