# 星云链Dapp开发(四):智能合约开发与部署

## 创建账户

```
./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:    // 这里填写账户密码
Repeat passphrase:   // 确认密码
```

账户创建成功以后会返回一个account地址，如下

```
Address: n1Q4o6dgaACp7MsRJbjKQeefZm1PRPCxNLG
```

新账户的keystore文件将会被放置在`$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`内

## 往新账户转账

由于发布合约的时候需要消耗gas，而我们新创建的账户并没有gas。系统初始的21个矿工每个有`5000000000000000000000000(5 * 10^24)表示5NAS`.这21个初始矿工的账户地址在根目录`conf/default/genesis.conf`配置文件中。我们选择地址:`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`给我们新创建的地址转币

- 检查账户状态

```
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"}'
```

如果返回：

```
{
  "result": {
    "balance": "5000000000000000000000000",
    "nonce": "0",
    "type": 87
  }
}
```

- 发送交易

```
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1Q4o6dgaACp7MsRJbjKQeefZm1PRPCxNLG", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'
```

注意:把这里`to`的地址换成你自己新创建的账户地址

返回如下结果

```
{
  "result": {
    "data": "CiD1/TTjXM7/sw5rFWiLRT5xGBmo+4iCUte29zvGy11oExIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXaMPE46mIdlX1Pzpo1uzIeZv/oq33OrdZIhAAAAAAAAAAAA3gtrOnZAAAKAEwmZDl3gU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBReuvyNEnpX0DWojGP7+KkA1rJpVAeVNutuPZP/otV8ZaKCVbICQjX7JnOy6vhipCQVkzpuxUG9D6uBMOsLsx7QE="
  }
}
```

然后，我们将签好名的交易原始数据提交到本地私有链里的星云节点，下面的data字段的内容就是上面产生的。

```
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiD1/TTjXM7/sw5rFWiLRT5xGBmo+4iCUte29zvGy11oExIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXaMPE46mIdlX1Pzpo1uzIeZv/oq33OrdZIhAAAAAAAAAAAA3gtrOnZAAAKAEwmZDl3gU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBReuvyNEnpX0DWojGP7+KkA1rJpVAeVNutuPZP/otV8ZaKCVbICQjX7JnOy6vhipCQVkzpuxUG9D6uBMOsLsx7QE="}'
```

返回如下执行结果:

```
{
  "result": {
    "txhash": "f5fd34e35cceffb30e6b15688b453e711819a8fb888252d7b6f73bc6cb5d6813",
    "contract_address": ""
  }
}
```

> 不论使用的哪一种方法发送交易，我们都会得到两个返回值，txhash和contract_address。其中txhash为交易hash，是一个交易的唯一标识。如果当前交易是一个部署合约的交易，contract_address将会是合约地址，调用合约时都会使用这个地址，是合约的唯一标识。

使用txhash我们可以查看交易收据，知道当前交易的状态。

```
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"f5fd34e35cceffb30e6b15688b453e711819a8fb888252d7b6f73bc6cb5d6813"}'
```

执行结果：

```
{
  "result": {
    "hash": "f5fd34e35cceffb30e6b15688b453e711819a8fb888252d7b6f73bc6cb5d6813",
    "chainId": 100,
    "from": "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE",
    "to": "n1Q4o6dgaACp7MsRJbjKQeefZm1PRPCxNLG",
    "value": "1000000000000000000",
    "nonce": "1",
    "timestamp": "1540966425",
    "type": "binary",
    "data": null,
    "gas_price": "1000000",
    "gas_limit": "2000000",
    "contract_address": "",
    "status": 2,
    "gas_used": "",
    "execute_error": "",
    "execute_result": "",
    "block_height": "0"
  }
}
```

这里的status可能有三种状态值，0，1和2。

- 0: 交易失败. 表示当前交易已经上链，但是执行失败了。可能是因为部署合约或者调用合约参数错误。
- 1: 交易成功. 表示当前交易已经上链，而且执行成功了。
- 2: 交易待定. 表示当前交易还没有上链。可能是因为当前交易还没有被打包；如果长时间处于当前状态，可能是因为当前交易的发送者账户的余额不够支付上链手续费。