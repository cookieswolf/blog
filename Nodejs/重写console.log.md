# 重写console.log

## 前端

```
const defineConsole = (DEBUG)=>{
    const console = ((Console)=>{
        if(DEBUG){
            return {
                log:(params)=>{
                    Console.log(params)
                },
                warn:(params)=>{
                    Console.warn(params)
                }
            }
        }else{
            return {
                log:()=>{},
                warn:()=>{},
                info:()=>{}
            }
        }
    })(window.console)
    
    window.console = console
}
```

## 参考文档

- [前端console.log的日志tips-上线](https://www.jianshu.com/p/7757f5cd197c)