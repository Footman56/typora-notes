github copliot 中支持mcp ,在idea中使用的时候就简单配置一下，顺便执行下mcp的配置以及生效流程

1. 前提
   1. 需要一些客户端支持mcp，比如copliot
   2. 需要本地安装node.js 服务。因为是通过node.js 中的npx 来获取对应的mcp服务的
2. 在copliot 中创建mcp 的配置文件，一般是json形式的
3. 在json文件中添加对应的服务器

```json
{
  "servers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp/",
      "requestInit": {
        "headers": {
          "Authorization": "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"
        }
      }
    },
    "iterm-mcp": {
      "command": "npx",
      "args": [
        "-y",
        "iterm-mcp"
      ]
    },
    "spec-workflow": {
      "command": "npx",
      "args": [
        "-y",
        "@pimzino/spec-workflow-mcp@latest",
        "/Users/peilizhi/javaProjects/xrxs-appraisal-springboot",
        "--AutoStartDashboard"
      ]
    }
  }
}
```

注意一下文件的格式，最外层是servers 为固定的，下一级为具体的服务，比如github、spec-workflow 服务。有了这些服务，那么在生成代码或者AI交互的时候就可以利用这些服务进行业务拓展 ，可以读取数据库、操作excel什么的，这些需要对应的服务

4. 重启idea 来实现mcp中服务的下载。这步很重要。



