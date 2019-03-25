# JS对域名测速

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>测速中</title>
</head>
<body>
    <p><a href='https://www.bootcdn.cn/' target='_blank'>www.bootcdn.cn</a>&nbsp;&nbsp;<span></span></p>
    <p><a href='https://github.com/yuanjunliang' target='_blank'>github.com/yuanjunliang</a>&nbsp;&nbsp;<span></span></p>
    <p><a href='https://www.jianshu.com/u/7048b08aa4df' target='_blank'>www.jianshu.com/u/7048b08aa4df</a>&nbsp;&nbsp;<span></span></p>

    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
    <script>
    var ping    = 1,
    urlList = ['https://www.bootcdn.cn/','https://github.com/yuanjunliang','https://www.jianshu.com/u/7048b08aa4df'];
 
    setInterval("ping++",100);
    newRequest();
    function newRequest(){
        for(var i=0;i<urlList.length;i++){
            $("p").eq(i).find('span').html('测速中...');
            $("p").eq(i).find('span').append("<img src="+urlList[i]+"/"+Math.random()+" width='1' height='1' onerror='autotest("+i+")' style='display:none'>");
        }
    }
    
    function autotest(i){
        $("p").eq(i).find("span").text(ping*100+"ms");
    }
    </script>
</body>
</html>
```