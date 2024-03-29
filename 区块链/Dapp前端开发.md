# 星云链Dapp开发(六):Dapp前端开发

## 需要用到的lib库

- [neb.js](https://github.com/nebulasio/neb.js)
- [nebPay](https://github.com/nebulasio/nebPay)
- [jQuery cnd](http://www.jq22.com/cdn/)

## 前端源码

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>星云链Dapp</title>
</head>
<body>
    <div>
        <h1>发布信息</h1>
        <div>标题：</div>
        <div><input type="text" id="title" style="width:370px"/></div>
        <div>内容：</div>
        <div><textarea cols="50" rows="10" id="content"></textarea></div>
        <div><input type="button" value="发布" id="publish"></div>
    </div>

    <div>
        <h1>执行结果：</h1>
        <div>
            <textarea cols="100" rows="5" id="result"></textarea>
        </div>
    </div>

    <br/>
    <hr/>

    <div>
        <h1>查询信息：</h1>
        <div>作者地址：</div>
        <div>
            <input type="text" id="author" style="width:370px"/>&nbsp;<input type="button" id="search" value="查询" />
        </div>
        <div>信息：</div>
        <div id="queryResult" style="height:100px;width:370px;border:1px solid #aaa">

        </div>
    </div>

    <div style="height:50px"></div>

    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/nebulas@0.5.5/dist/nebulas.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/nebpay.js@0.2.1/dist/nebPay.min.js"></script>
    <script>
        "use strict";

        // 合约地址
        var dappAddress = "n1puot9ACpKANFkgchMNT1xd33ZzsmMKoNs";

        // 直接访问星云链的api
        var nebulas = require("nebulas"),
            Account = nebulas.Account,
            neb = new nebulas.Neb();
        // 设置使用的网络
        neb.setRequest(new nebulas.HttpRequest("http://0.0.0.0:8685"));

        // NebPay SDK 为不同平台的交易提供了统一的支付接口
        // 开发者在Dapp页面中使用NebPay API可以通过浏览器插件钱包、手机app钱包等实现交易支付和合约调用。
        var NebPay = require("nebpay");
        var nebPay = new NebPay();

        // 执行合约返回的交易流水号，用于查询交易信息
        var serialNumber;

        $("#publish").click(function () {
            var title = $("#title").val().trim();
            var content = $("#content").val().trim();

            if (title === "" || content === "") {
                alert("标题或内容不能为空！");
                return;
            }

            // 执行合约中的save方法
            serialNumber = nebPay.call(dappAddress, "0", "save", "[\"" + title + "\",\"" + content + "\"]", {
                listener : function (resp) {
                    console.log(resp);
                    // 清空信息
                    $("#title").val("");
                    $("#content").val("");

                    // 延迟5秒执行
                    intervalQuery = setInterval(function () {
                        queryResultInfo();
                    }, 5000);
                }
            });
        });
        // 定时器
        var intervalQuery;
        // 根据交易流水号查询执行结果数据
        function queryResultInfo() {
            nebPay.queryPayInfo(serialNumber)
                .then(function (resp) {
                    $("#result").val(resp);
                    var respObject = JSON.parse(resp)
                    if(respObject.code === 0){
                        alert(`提交信息成功!`);
                        clearInterval(intervalQuery);
                    }
                })
                .catch(function (err) {
                   console.log(err);
                })
        }

        // 查看信息
        $("#search").click(function () {
            var author = $("#author").val().trim();
            if (author === "") {
                alert("作者地址不能为空！");
                return;
            }

            var from = Account.NewAccount().getAddressString();
            var value = "0";   // 金额
            var nonce = "0";   // 交易序号
            var gas_price = "1000000" // 手续费价格
            var gas_limit = "2000000" // 手续费限制
            var callFunction = "read";
            var callArgs = "[\"" + author + "\"]"; //in the form of ["args"]
            var contract = { // 合约
                "function": callFunction,  // 方法名
                "args": callArgs            // 参数
            }

            // 使用api.call执行合约中的方法
            neb.api.call(from, dappAddress, value, nonce, gas_price, gas_limit, contract).then(function (resp) {
                var ressultObj = JSON.parse(resp.result);
                $("#result").val(resp.result);

                var title = ressultObj.title;
                var content = ressultObj.content;
                var time = new Date(ressultObj.timestamp);

                $("#queryResult").html("标题：" + title + "<br/>" + "内容：" + content + "<br/>" + "时间：" + time);

            }).catch(function (err) {
                console.log("error:" + err.message)
            })
        });
    </script>
</body>
</html>
```