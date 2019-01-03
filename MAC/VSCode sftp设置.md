# VSCode SFTP配置

- 安装`sftp`插件
- 按`F1`调取插件配置文档
- 输入以下配置

```
{
    "host": "47.75.111.149",
    "port": 22,
    "username": "bottos",
    "password": "Bottos201804",
    "protocol": "sftp",
    "agent": null,
    "privateKeyPath": null,
    "passphrase": null,
    "passive": false,
    "interactiveAuth": false,
    "remotePath": "/home/bottos/go/src/github.com/xingyun/go-nebulas",
    "uploadOnSave": true,
    "syncMode": "update",
    "watcher": {
        "files": false,
        "autoUpload": false,
        "autoDelete": false
    },
    "ignore": [
        "**/.vscode/**",
        "**/.git/**",
        "**/.DS_Store"
    ]
}
```