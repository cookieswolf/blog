# IPFS


## IFPS 文件加密上传下载(浏览器端)

```
import IPFS from 'ipfs'
import cryptojs from 'crypto-js'
import ab2str from "arraybuffer-to-string"
import randomBytes from 'randombytes'


/**
 * 上传文本内容到IPFS
 * @param {*string} context IPFS上加密文件路径
 * @param {*string} secretKey 加密密钥
 * @param {*function} callback 
 */
const upload = ({context},callback)=>{
    let node = new IPFS()
    node.on("ready",()=>{
        // 加密文件
        let password = getPassword()
        let cryptoBuf = aesEncrypto(context,password)
        node.add([Buffer.from(cryptoBuf)],(error,response)=>{
            if(error){
                callback(error,null)
            }else{
                let fileObj = response[0]
                // let secretPath = aesEncrypto(fileObj.path,password)
                callback(null,{secretPath:fileObj.path,password})
            }
        })
    })
}

/**
 * 读取IPFS上的文本内容
 * @param {string} secretPath IPFS上加密文件路径
 * @param {string} password 加密密钥
 * @param {function} callback 回调函数
 */
const download = ({filePath,password},callback)=>{
    let node = new IPFS()
    node.on("ready",()=>{
        node.cat(filePath,(error,response)=>{
            console.log({error,response})
            if(error){
                callback(error,null)
            }else{
                let context = aesDecrypto(response,password)
                callback(null,context)
            }
        })
    })
}

/**
 * IPFS文件上传
 * @param {*File} file 要上传的文件
 * @param {*function} callback 回调函数
 */
function uploadFile(file,callback){
    let reader = new FileReader();
    reader.readAsArrayBuffer(file);
    reader.onloadend = function(e) {
        let fileBuf = this.result
        let node = new IPFS()
        node.on("ready",()=>{
            // 文件加密
            let password = getPassword()
            let cryptoBuf = aesEncrypto(buf2hex(fileBuf),password)
            node.add([node.types.Buffer.from(cryptoBuf)],(error,response)=>{
                if(error){
                    callback(error,null)
                }else{
                    let filePath = response[0].path
                    callback(null,{filePath,password})
                }
            })
        })
    };  
}

/**
 * IPFS文件下载
 * @param {*string} filePath 文件路径
 * @param {*string} filename 文件名称
 */
function downloadFile({filePath,filename,password}){
    let node = new IPFS()
    node.on("ready",()=>{
        node.cat(filePath,(error,response)=>{
            if(error){
                // callback(error,null)
                console.error(error)
            }else{
                let fileContent = aesDecrypto(ab2str(response),password)
                downloadFileWithContent(filename,str2buf(fileContent))
            }
        })
    })
}

/**
 * 文件下载
 * @param {*string} filename 文件名称
 * @param {*Buffer} content 文件内容
 */
function downloadFileWithContent(filename,content) {
    // 创建隐藏的可下载链接
    var eleLink = document.createElement('a');
    eleLink.download = filename;
    eleLink.style.display = 'none';
    // 字符内容转变成blob地址
    var blob = new Blob([content]);
    eleLink.href = URL.createObjectURL(blob);
    // 触发点击
    document.body.appendChild(eleLink);
    eleLink.click();
    // 然后移除
    document.body.removeChild(eleLink);
};


/**
 * aes encrypto
 * @param {* string} message 
 * @param {* string} secretKey 
 */
const aesEncrypto = (message,secretKey)=>{
    return cryptojs.AES.encrypt(message, secretKey).toString();
}

/**
 * aes decrypto
 * @param {* buffer} aesSecretMessage 
 * @param {* string} secretKey 
 */
const aesDecrypto = (aesSecretMessage, secretKey)=>{
    let bytes  = cryptojs.AES.decrypt(aesSecretMessage, secretKey);
    let plaintext = bytes.toString(cryptojs.enc.Utf8);
    return plaintext;
}

/**
 * buffer type to string
 * @param {* buffer} buffer 
 */
var buf2hex = function buf2hex(buffer) {
    return Array.prototype.map.call(new Uint8Array(buffer), function (x) {
        return ('00' + x.toString(16)).slice(-2);
    }).join('');
};

/**
 * string type to buffer
 * @param {* string} str 
 * @param {* hex || base64} enc 
 */
var str2buf = function str2buf(str) {
    var enc = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : "hex";

    if (!str || str.constructor !== String) return str;
    if (!enc && undefined.isHex(str)) enc = "hex";
    if (!enc && undefined.isBase64(str)) enc = "base64";
    return Buffer.from(str, enc);
};

/**
 * 随机生成密码
 */
const getPassword = ()=>{
    return randomBytes(32).toString("hex");
}


export default {
    upload,
    download,
    uploadFile,
    downloadFile
}
```