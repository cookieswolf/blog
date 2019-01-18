# WebSocket简单封装


```
export default class SocketTool{
    constructor(config){
        this.config = config
        this.init()
    }

    init(){
        this.websock = new WebSocket(this.config.url);
        this.timerId = 0
        this.websock.addEventListener('open',this.onopen.bind(this))
        this.websock.addEventListener('message',this.onmessage.bind(this))
        this.websock.addEventListener('error',this.onerror.bind(this))
        this.websock.addEventListener('close',this.onclose.bind(this))
    }

    onmessage(event){
        console.log("onmessage")
    }

    onopen(){
        console.log("onopen")
        this.keepAlive()
    }

    onerror(){
        console.log("onerror")
    }

    onclose(){
        console.log("onclose")
        this.cancelKeepAlive()
        this.websock = new WebSocket(config.url);
    }

    // checkReadyState(){
    //     switch (this.websock.readyState) {
    //         case WebSocket.CONNECTING:
    //           // do something
    //           console.log("connecting")
    //           break;
    //         case WebSocket.OPEN:
    //           // do something
    //           console.log("open")
    //           break;
    //         case WebSocket.CLOSING:
    //           // do something
    //           console.log("closing")
    //           break;
    //         case WebSocket.CLOSED:
    //           // do something
    //           console.log("closed")
    //           break;
    //         default:
    //             console.log("default")
    //           // this never happens
    //           break;
    //       }
    // }

    keepAlive(){
        console.log("keepAlive")
        let timeout = this.config.timeout || 2000;  
        if (this.webSocket.readyState == WebSocket.OPEN) {  
            this.webSocket.send('');  
        }  
        this.timerId = setTimeout(keepAlive, timeout);  
    }

    cancelKeepAlive() {  
        console.log("cancelKeepAlive")
        if (this.timerId) {  
            clearTimeout(this.timerId);  
        }  
    }
}

```


参考文章:

- [处理 Websocket 超时问题](http://www.jstips.co/zh_cn/javascript/working-with-websocket-timeout/)
- [WebSocket 教程](http://localhost:9999/#/home)