这个Readme包含了使用 Apex Legends Live API 的基本信息

# 概述

Live API 是一个内置机制，用于监听游戏事件并通过编程与游戏进行交互。它提供了一个稳定的接口，允许应用开发人员与 Apex Legends 集成，并基于游戏事件驱动其他服务。这个 API 特别适用于在线直播、Live比赛，甚至是比赛结束后的离线分析。

要使用Live API 提供游戏事件，必须允许游戏连接的服务器，并且只能通过使用额外的命令行参数 `+cl_liveapi_enabled 1` 来激活。

# 快速入门

该 API 是基于 WebSockets 和 Google 的 Protocol Buffer (protobuf) 技术构建的。Live API 支持的所有游戏事件和请求都在 `events.proto` protobuf 文件中有记录，该文件位于本目录中。如果您对 protobuf 不熟悉，请访问 https://developers.google.com/protocol-buffers 了解更多信息。

一旦获得 Protobuf 编译器 `protoc`，您可以轻松地为您选择的语言生成 Protobuf 绑定。例如，要为 Python 生成绑定，请使用以下命令：

`protoc.exe --proto_path=<当前目录> --python_out=<项目输出目录> events.proto`

然后，可以将这些绑定包含在您的应用代码中。为了接收事件，您的应用程序必须打开一个 WebSocket 服务器，游戏可以连接并向其发送事件。游戏还必须在附加的命令行参数下运行，以激活 LiveAPI 并连接到应用程序。例如，如果应用程序在本机的端口 7777 上具有 WebSocket 服务器，则可以将以下命令行传递给游戏：

`+cl_liveapi_enabled 1 +cl_liveapi_ws_servers "ws://127.0.0.1:7777"`

使用这些参数启动游戏后，游戏将连接到本地主机上的端口 7777 的 WebSocket 服务器。游戏连接到服务器后（并且服务器允许使用 Live API），游戏事件将发送到 WebSocket 服务器应用程序。

# 配置

Live API 有几个选项可以通过命令行或单独的 JSON 配置文件（名为 config.json）进行指定，该文件位于 Windows 用户的 "Saved Games" 文件夹中（默认位置为 %USERPROFILE%\Saved Games）。

一个简单的 `config.json` 文件位于此文件夹中供参考。要使用 JSON 配置文件，只需将其放置在 "Saved Games" 文件夹下的 Respawn\Apex\assets\temp\live_api 目录中。例如，对于名为 "Nessie" 的用户，其 Saved Games 文件夹位于 `C:\Users\Nessie\Saved Games`，则应将 `config.json` 文件移动到 `C:\Users\N

essie\Saved Games\Respawn\Apex\assets\temp\live_api\config.json`。

如前所述，也可以使用命令行参数来配置Live API。这些参数将覆盖 config.json 文件中的任何配置值。可以将以下一些命令行参数传递给游戏：

	cl_liveapi_enabled（默认值为 "0"）
	   启用Live API 功能。

	cl_liveapi_allow_requests（默认值为 "1"）
	   允许处理和运行任何传入的请求。

	cl_liveapi_pretty_print_log（默认值为 "0"）
	   使 JSON 输出更易读。

	cl_liveapi_use_protobuf（默认值为 "1"）
	   使用 protobuf 作为序列化格式。否则，使用 JSON。

	cl_liveapi_ws_keepalive（默认值为 "30"）
	   发送 Pong 给任何连接的服务器的时间间隔。

	cl_liveapi_ws_retry_count（默认值为 "5"）
	   在将连接标记为不可用之前尝试连接的次数。

	cl_liveapi_ws_retry_time（默认值为 "30"）
	   重试之间的时间间隔。

	cl_liveapi_ws_servers（默认值为 ""）
	   要连接的地址列表，以逗号分隔。必须采用 'ws://domain.com:portNum' 的形式。

	cl_liveapi_ws_timeout（默认值为 "300"）
	   WebSocket 连接超时时间（以秒为单位）。
	   
	cl_liveapi_ws_lax_ssl（默认值为 "1"）
	   跳过所有 WSS 连接的 SSL 证书验证。允许使用自签名证书。
	   
	cl_liveapi_ws_event_delay（默认值为 ""）
	   广播事件到所有连接时使用的延迟时间（以秒为单位）。
	   
	cl_liveapi_requests_psk（默认值为 ""）
	   用于验证请求的预共享密钥。设置后，如果接收到的请求的密钥不匹配，则请求将被拒绝。
		
	cl_liveapi_requests_psk_tries（默认值为 "10"）
	   在使用错误的密钥进行请求时允许的尝试次数，超过这个次数将关闭连接。最小值为 10 次。
		
	cl_liveapi_session_name（默认值为 ""）
	   可用于识别此客户端的 WebSocket 连接中的会话名称。

# 通过 WebSocket 发送和接收消息

`events.proto` 文件描述了 LiveAPI 使用的所有消息，并在每个定义的消息上添加了有关其行为的附加注释。虽然 LiveAPI 主要用于监听游戏内事件，但 WebSockets 允许应用程序与游戏之间进行双向通信。

为了简化消息的读取和接收，LiveAPI 提供了两种类型的信封消息：`LiveAPIEvent` 和 `Request`。顾名思义，`Request` 消息是 WebSocket 服务器应用程序发送的消息，以便让游戏执行某个操作。`LiveAPIEvent` 是游戏发送给任何 WebSocket 连接的所有消息的信封。

请注意，当

将 JSON 作为主要的数据传输方法时，`LiveAPIEvent` 消息是可读的，并且不包装在 LiveAPIEvent 信封中。这是为了与旧应用程序的向后兼容性。使用 JSON 进行的 LiveAPI 请求仍然必须通过 `Request` 消息类型发送。建议使用 Protobuf 绑定，以确保正确进行序列化，即使使用 JSON。

在使用 Protobuf 和 WebSockets 时，正确解包/打包消息非常重要。要在 protobuf 中读取 `LiveAPIEvent` 消息，需要获取 `gameMessage` 的类型，并使用该类型的实例调用解包函数。在继续之前，请熟悉 proto3 `Any` 字段类型，网址为 https://protobuf.dev/programming-guides/proto3/#any，并确保查阅所使用编程语言的文档。

要生成一个 protobuf 的 `Request` 消息，必须指定一个动作，并适当填充该消息的任何字段。请查阅有关使用 oneof 字段的 protobuf 文档，网址为 https://protobuf.dev/programming-guides/proto3/#oneof。LiveAPI 可以在收到请求后确认成功的请求。这些请求会以 `Response` 消息发送，仅发送到发起请求的连接。请注意，收到响应并不意味着请求已完成。这是开发过程中的一个很好的工具，可确保请求编写正确。

最后，某些请求在成功完成后可能会触发游戏事件。例如，具有 `changeCam` 动作的 `Request` 将导致一个带有 `ObserverSwitched` gameMessage 的 `LiveAPIEvent` 作为有效载荷。

# 支持安全的 WebSocket（wss）

Live API 支持安全的 WebSocket（wss），以避免在明文中传输敏感数据。打算使用 WSS 的应用程序必须监听端口 443，因为这是唯一可接受的用于安全协商的端口。默认情况下，存在宽松的 SSL 策略，允许使用各种证书，包括自签名证书。这是为了方便集成，但可能不适用于生产环境或网络通道不受信任的情况（例如，通过互联网或本地数据中心连接）。当禁用宽松的 SSL 策略时，只有来自 VeriSign 和 DigiCert CA 的证书才能适当验证，否则安全连接将失败。建议与 WSS 一起使用预共享密钥功能（cl_liveapi_requests_psk），以确保请求始终由受信任的方发送。

# 常见问题

- 如何向游戏传递命令行参数？
在 Steam 上，导航到游戏库，右键单击 Apex Legends。在“常规”选项中，将所需的命令行添加到“启动选项”文本

框中。

- 如何创建 WebSocket 服务器？
要创建 WebSocket 服务器，您可以使用您喜欢的编程语言和库。许多语言都有成熟的 WebSocket 库可供使用，例如 `websockets`（Python），`ws`（Node.js）和 `websocket`（Java）。选择一个适合您的语言和应用程序需求的库，并按照其文档进行设置和使用。

- 如何处理接收到的游戏事件？
您的 WebSocket 服务器应用程序需要解析和处理接收到的游戏事件。根据您选择的编程语言和库，您可能需要使用相应的功能来解包和处理 Protobuf 消息。确保阅读所使用库的文档和示例代码，以便正确处理游戏事件。

- 如何发送请求给游戏？
要向游戏发送请求，您需要创建一个符合 `Request` 消息结构的 Protobuf 消息，并将其发送到游戏连接的 WebSocket。确保根据所使用的编程语言和库的要求，正确序列化和发送消息。

这是关于 Apex Legends Live API 的基本信息。请注意，此信息基于先前的知识，实际的Live API 可能会有所变化。因此，在使用 API 时，请参考官方文档和资源，以确保您具有最新和准确的信息。
