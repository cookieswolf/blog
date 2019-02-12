# Restful Api Document

## Swagger

### 了解Swagger
Swagger是一个规范和完整的框架，用于生成、描述、调用和可视化RESTful风格的Web服务。Swagger的目标是对REST API定义一个标准的和语言无关的接口，可让人和计算机无需访问源码、文档或网络流量监测就可以发现和理解服务的能力。当通过Swagger进行正确定义，用户可以理解远程服务并使用最少实现逻辑与远程服务进行交互。

### Swagger应用：
Swagger可以生成一个交互性的API控制台，开发者可以用来快速学习API，同时也方便测试人员了解API。
Swagger支持OpenApi规范生成代码，生成的客户端和服务器端框架代码可以加速开发和测试速度。
Swagger文件可以用不同的编程语言的代码注释中自动生成。
Swagger文档可作为客户产品文档的一部分进行发布，可用于项目内部API审核，支持API自动生成同步的在线文档。

### swagger项目
Swagger是一组开源项目，其中主要要项目如下： 
- Swagger-ui：这是一套 HTML/CSS/JS 框架用于解析遵守 Swagger spec 的 JSON 或 YML 文件，可以为Swagger兼容API动态生成文档。

- Swagger-editor：这是在线编辑器，用于验证你的 YML/JSON 格式的内容是否违反Swagger spec 。Swagger spec提供了一个方法，使我们可以用指定的JSON/YAML摘要来描述你的API。可让使用者在浏览器里以YAML格式编辑Swagger API规范并实时预览文档。可以生成有效的Swagger JSON描述，并用于所有Swagger工具中。

- Swagger-codegen：一个模板驱动引擎，通过分析用户Swagger资源声明，这个工具可以为不同的平台生成客户端 SDK（比如 Java、JavaScript、Python 等）。这些客户端代码帮助开发者在一个规范平台中整合 API ，并且提供了更多健壮的实现，可能包含了多尺度、线程，和其他重要的代码。

- Swagger-core: 用于Java/Scala的的Swagger实现。与JAX-RS(Jersey、Resteasy、CXF…)、Servlets和Play框架进行集成。

- Swagger-js: 用于JavaScript的Swagger实现。

- SwaggerHub：是 Swagger API 的一个集成服务网站，提供 Swagger 的企业级服务需求，集成 Github OAuth，一键将 - Swagger 发布到 Github等便利平台。 
swaggerhub在线平台入口：https://app.swaggerhub.com/home
swaggerhub帮助文档：https://app.swaggerhub.com/help/index

- Swagger Inspector：测试API和生成OpenAPI的开发工具，使用Swagger Inspector测试的API可以生成到SwaggerHub中。Swagger Inspector的建立是为了解决开发者的三个主要目标：1. 执行简单的API测试；2. 生成OpenAPI文档；3. 探索新的API功能。 
inspector帮助文档：https://swagger.io/docs/swagger-inspector/how-to-use-swagger-inspector/


## Apidoc修改成json请求传参

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

## jsdoc-to-markdown使用

```
jsdoc2md "src/**/*.js" > api.md  // 把整个目录下的注释生成api.md
```

## 其他文档生成工具

- [Read the Docs](https://readthedocs.org/)
- [API文档](https://api-docs.io/)
- [Swagger](https://swagger.io/)
- [swagger 小试牛刀](https://blog.csdn.net/IT_faquir/article/details/82466103)
- [doxmate](https://github.com/JacksonTian/doxmate)
- [JSDOC](https://github.com/jsdoc3/jsdoc)
- [markdown-api](https://github.com/davidfig/markdown-api#readme)
- [JSDOC中文文档](https://www.css88.com/doc/jsdoc/index.html)
- [jsdoc-to-markdown](https://github.com/jsdoc2md/jsdoc-to-markdown/blob/master/docs/API.md)
- [【ApiDoc】自动化导出接口文档之HTML/Markdown/PDF实践](https://www.jianshu.com/p/16ecf8a408e8)