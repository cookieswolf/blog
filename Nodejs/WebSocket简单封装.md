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

## websocket-heartbeat-js源码

```
/**
 * `WebsocketHeartbeatJs` constructor.
 *
 * @param {Object} opts
 * {
 *  url                  websocket链接地址
 *  pingTimeout 未收到消息多少秒之后发送ping请求，默认15000毫秒
    pongTimeout  发送ping之后，未收到消息超时时间，默认10000毫秒
    reconnectTimeout
    pingMsg
 * }
 * @api public
 */

function WebsocketHeartbeatJs({
    url, 
    pingTimeout = 15000,
    pongTimeout = 10000,
    reconnectTimeout = 2000,
    pingMsg = 'heartbeat'
}){
    this.opts ={
        url: url,
        pingTimeout: pingTimeout,
        pongTimeout: pongTimeout,
        reconnectTimeout: reconnectTimeout,
        pingMsg: pingMsg
    };
    this.ws = null;//websocket实例

    //override hook function
    this.onclose = () => {};
    this.onerror = () => {};
    this.onopen = () => {};
    this.onmessage = () => {};
    this.onreconnect = () => {};

    this.createWebSocket();
}
WebsocketHeartbeatJs.prototype.createWebSocket = function(){
    try {
        this.ws = new WebSocket(this.opts.url);
        this.initEventHandle();
    } catch (e) {
        this.reconnect();
        throw e;
    }     
};

WebsocketHeartbeatJs.prototype.initEventHandle = function(){
    this.ws.onclose = () => {
        this.onclose();
        this.reconnect();
    };
    this.ws.onerror = () => {
        this.onerror();
        this.reconnect();
    };
    this.ws.onopen = () => {
        this.onopen();
        //心跳检测重置
        this.heartCheck();
    };
    this.ws.onmessage = (event) => {
        this.onmessage(event);
        //如果获取到消息，心跳检测重置
        //拿到任何消息都说明当前连接是正常的
        this.heartCheck();
    };
};

WebsocketHeartbeatJs.prototype.reconnect = function(){
    if(this.lockReconnect || this.forbidReconnect) return;
    this.lockReconnect = true;
    this.onreconnect();
    //没连接上会一直重连，设置延迟避免请求过多
    setTimeout(() => {
        this.createWebSocket();
        this.lockReconnect = false;
    }, this.opts.reconnectTimeout);
};
WebsocketHeartbeatJs.prototype.send = function(msg){
    this.ws.send(msg);
};
//心跳检测
WebsocketHeartbeatJs.prototype.heartCheck = function(){
    this.heartReset();
    this.heartStart();
};
WebsocketHeartbeatJs.prototype.heartStart = function(){
    if(this.forbidReconnect) return;//不再重连就不再执行心跳
    this.pingTimeoutId = setTimeout(() => {
        //这里发送一个心跳，后端收到后，返回一个心跳消息，
        //onmessage拿到返回的心跳就说明连接正常
        this.ws.send(this.opts.pingMsg);
        //如果超过一定时间还没重置，说明后端主动断开了
        this.pongTimeoutId = setTimeout(() => {
            //如果onclose会执行reconnect，我们执行ws.close()就行了.如果直接执行reconnect 会触发onclose导致重连两次
            this.ws.close();
        }, this.opts.pongTimeout);
    }, this.opts.pingTimeout);
};
WebsocketHeartbeatJs.prototype.heartReset = function(){
    clearTimeout(this.pingTimeoutId);
    clearTimeout(this.pongTimeoutId);
};
WebsocketHeartbeatJs.prototype.close = function(){
    //如果手动关闭连接，不再重连
    this.forbidReconnect = true;
    this.heartReset();
    this.ws.close();
};
if(window) window.WebsocketHeartbeatJs = WebsocketHeartbeatJs;
export default WebsocketHeartbeatJs;
```


参考文章:

- [处理 Websocket 超时问题](http://www.jstips.co/zh_cn/javascript/working-with-websocket-timeout/)
- [WebSocket 教程](http://localhost:9999/#/home)

- [Socket.IO打造基础聊天室](https://www.jianshu.com/p/51b0d1f80392)
- [socket.io 的详细工作流程是怎样的？](https://www.zhihu.com/question/31965911)
- [Socket.IO进阶](https://hongtoushizi.iteye.com/blog/2019517)
- [node.js – 从socket.io中的客户端控制心跳超时](https://codeday.me/bug/20181022/318818.html)
- [websocket-heartbeat-js](https://github.com/zimv/websocket-heartbeat-js)
- [初探和实现websocket心跳重连(npm: websocket-heartbeat-js)](http://www.cnblogs.com/1wen/p/5808276.html)