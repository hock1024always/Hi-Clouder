# **为什么需要MCP？**

## **LLM 现状**

我们都知道`LLM只能预测文本，仅仅限于回答，而不做事情`。比如让LLM发邮件或者写代码，LLM只会给出一个步骤，而不会帮我们去实现。![在这里插入图片描述](./图片/MCP1)如果需要LLM能做具体的事情，不只是口头说说，比如实际去发邮件，就需要用到连接工具Tools，比如发邮件的工具。`将LLM的输出内容，输入到Email Tool工具中进行发邮件的操作`。![er](./图片/MCP2)这样我们的AI才更像是贾维斯(钢铁侠中的人工智能)，能帮我们做事情的，而不仅仅教我们如何做事情的。如果我们不仅仅需要LLM发邮件，而是做一些其他的事情，比如记一个Todolist，此时就也需要另一个Tool。`将LLM的输出内容，输入到Todolist Tool工具中进行记录Todo的操作`。![在这里插入图片描述](./图片/MCP3)但是这也仅仅是 FanOne 同学的做法，也就是说在 LLM 和 Tool 交互的这段协议是只有 FanOne 知道，`别人如果想要用FanOne同学写的 Email Tools 来发邮件，就要使用FanOne写的协议来连接 LLM 和 Tool。`

## **Tool 现状**

此时FanTwo同学觉得这段FanOne同学写的协议不好，很臃肿，代码逻辑不强，导致发邮件的时候经常出错，`于是自己写了一套更好、更快的协议`。![在这里插入图片描述](./图片/MCP4)但是FanThree也觉得FanTwo协议也不好，于是又开始一个循环，再次创建新的协议.... 这样会导致越来越多的连接协议出现。`做一个协议的代价是很大的，既要兼顾LLM的输出输入，也要兼顾Tool的输出输入。`

`为了规范LLM到Tools协议的连接，MCP就出现了`，只要大家都遵守这个协议，做的工具也就能直接复用，FanOne同学基于MCP协议做的Email Tool，FanTwo只要了解过MCP就知道如何使用Email Tool了

![在这里插入图片描述](./图片/MCP5)

## **类比**

有点像我们今天的蓝牙协议，USB协议之类的，通过蓝牙协议让各种无线设备进行连接，通过统一的USB接口协议，用一根线就能连接各种设备。`通过一个约定俗成的对接协议，统一输入输出源，抽象各种AI应用，防止重复造轮子。`![在这里插入图片描述](./图片/MCP6)我们明白MCP其实是一个协议之后，我们再来讲讲MCP的原理。

# **MCP 原理是什么？**

MCP 全称 Model Context Protocol，模型上下文协议，是 Anturopic 公司在2024年11月推出的一种开放协议，`旨在统一规范大语言模型与外部数据源和工具之间的通信。`![在这里插入图片描述](./图片/MCP7)

## **MCP 组件分析**

MCP 基于CS架构，由多个组件构成

- Host：宿主程序，用户`直接交互的桌面应用程序`，一般就是我们的PC电脑服务器。
- MCP Client：MCP客户端，在主机Host和MCP Server中保持连接，比如Claude客户端，Chatbox，Cline这些应用程序都内置了MCP Client，能配置MCP Server。
- Local Data Source：本地数据源，比如本地redis、mysql之类的。

![在这里插入图片描述](./图片/MCP8)读写本地资源

- Remote Service：远程服务，比如远程的github、gitlab之类的。

![在这里插入图片描述](./图片/MCP9)读写远程资源

## **MCP Server**

MCP Server 通过MCP协议连接，提供功能给 Client，可以提供主要的类型功能，分成两种类型：

- command：需要在本地启动一个MCP Server进程，`Client 和 Server 通过 stdio（标准输入输出）传输协议进行交互`，主要是实现一些本地执行的功能，比如数据库读取，操作系统命令，文件操作等等。

![在这里插入图片描述](./图片/MCP10)stdio

- SSE：Server-Send Events `Client和远端的Server通过SSE传输协议进行交互`。基于HTTP，通过长连接的方式持续获取消息。也就是客户端建立TCP链接后，向服务端发起一个HTTP请求，`服务端接收到请求后把要返回的内容，按照事件流的方式，不断推送给客户端`。跟下载文件一样，所有内容推送完了，连接才关闭。

![在这里插入图片描述](./图片/MCP11)SSE

目前社区已经有很多的开源 MCP Server了 mcp server examples

## **其他概念**

- Resources 资源：可以被客户端读取的静态资源，类文件数据，比如日志、数据库、图像等等。

![在这里插入图片描述](./图片/MCP12)本地资源

- Tools 工具：**可以被LLM调用的工具，但需要用户批准**，每个Tool都`由一个唯一标识的名称，具体描述以及功能参数组成。` Tool 使得模型可以与外部系统交互，通过调用API或者执行计算。

![在这里插入图片描述](./图片/MCP13)Tool 访问远程服务资源

```go
{
  name: string;          // 工具的唯一标识的名称
  description?: string;  // 具体描述
  inputSchema: {         // 工具的输入参数
    type: "object",
    properties: { ... }  // 工具特定的输入参数
  }
}
```

- Prompts 提示： 预先编写的提示词模版，帮助用户完成特定的任务

```go
{
  name: string;              // 提示词的唯一标识
  description?: string;      // 具体描述
  arguments?: [              // 可选的参数
    {
      name: string;          // 参数标识名称
      description?: string;  // 参数描述
      required?: boolean;    // 参数是否必须
    }
  ]
}
```

## **基础协议 & 数据结构**

MCP 是采用`JSON RPC 2.0` 来进行通信的。

- Requests 请求数据结构：

```go
interface Request {
  method: string;
  params?: { ... };
}
```

- Results 成功的响应数据结构

```go
interface Result {
  [key: string]: unknown;
}
```

- Errors 失败的响应结果数据结构

```go
interface Error {
  code: number;
  message: string;
  data?: unknown;
}
```

![在这里插入图片描述](./图片/MCP14)

# **MCP 怎么用？**

我们了解完MCP之后，我们再来看看如何使用MCP。根据上文介绍我们知道，需要以下：

1. MCP Client：这里就使用Cline了，因为vscode中插件就有。
2. MCP Server：平时Go用的多，本文就基于Go语言实现一个`发邮件的Server吧`。

## **配置MCP Client**

下载vscode，并且在插件中找到 Cline![在这里插入图片描述](./图片/640)点击上方配置MCP Server，我们可以使用第三方的，也可以使用自己编写的。![在这里插入图片描述](./图片/641)点击 installed 进行配置

![在这里插入图片描述](./图片/642)![在这里插入图片描述](./图片/643)点击Done即可，这个服务地址就是我们编写的MCP Server，下面我们来讲讲MCP Server的构建

## **编写MCP Server**

新建一个MCP Server，并添加对应的工具

```go
// 定义服务
s := server.NewMCPServer("Email Sender", "1.0.0") // 定义服务
tool := mcp.NewTool("hello_world",                // 定义工具
 mcp.WithDescription("Say hello to someone"),
 mcp.WithString("email", mcp.Required(), mcp.Description("The email address to send")),
 mcp.WithString("content", mcp.Required(), mcp.Description("The email content to send")),
)
s.AddTool(tool, emailHandler) // 添加工具
if err := server.ServeStdio(s); err != nil {
 fmt.Printf("Server error: %v\n", err)
}
```

具体的校验函数和发送邮箱逻辑

```go
func emailHandler(ctx context.Context, request mcp.CallToolRequest) (*mcp.CallToolResult, error) {
// 校验参数
 email, ok := request.Params.Arguments["email"].(string)
if !ok {
returnnil, errors.New("email must be a string")
 }
 content, ok := request.Params.Arguments["content"].(string)
if !ok {
returnnil, errors.New("content must be a string")
 }
// 具体的发送逻辑
 emailSender := NewEmailSender()
 err := emailSender.Send(email, content, content)
if err != nil {
returnnil, err
 }
return mcp.NewToolResultText(fmt.Sprintf("Send %s with content about %s Successfully", email, content)), nil
}
```

我们只需要执行 `go build main.go` 获取可执行文件，这个可执行文件，就是上面配置的服务地址。

具体的email发送逻辑就不过多展开了，源码在github上`https://github.com/CocaineCong/mcp-server-email` ，感兴趣的同学可以试试。

## **实验**

当我们让LLM发邮件的时候![在这里插入图片描述](./图片/645)LLM获取了我们的批准后便开始发邮件。![在这里插入图片描述](./图片/646)发送成功的返回![在这里插入图片描述](./图片/647)效果：![在这里插入图片描述](./图片/648)

# **参考**

[1] https://modelcontextprotocol.io/quickstart/server 

[2] https://github.com/mark3labs/mcp-go