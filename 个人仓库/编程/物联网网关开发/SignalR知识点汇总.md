## 一、SignalR的结构组成
1、服务端：一般是WebApi等后台服务且唯一
2、客户端：形式不限，且数量不限

## 二、代码组成部分
### 1、服务端
1. 需要在服务端继承Hub创建事件集线器
```csharp
/*在HUB中的自定义方法可被客户端进行调用*/
public class ThirdDeviceHub : Hub
{
    private readonly IMessagePushService _messagePushService;

    public ThirdDeviceHub(IMessagePushService messagePushService)
    {
        _messagePushService = messagePushService;
    }

    /// <summary>
    /// 客户端订阅设备组
    /// </summary>
    /// <param name="deviceGroup">设备组名</param>
    public async Task SubscribeToDeviceGroup(string deviceGroup)
    {
        // 将客户端连接加入设备组
        await Groups.AddToGroupAsync(Context.ConnectionId, deviceGroup);
        await Clients.Group(deviceGroup).SendAsync("Notify", $"Client {Context.ConnectionId} has joined the group {deviceGroup}");
    }

    /// <summary>
    /// 客户端取消订阅设备组
    /// </summary>
    /// <param name="deviceGroup">设备组名</param>
    public async Task UnsubscribeFromDeviceGroup(string deviceGroup)
    {
        // 将客户端连接从设备组中移除
        await Groups.RemoveFromGroupAsync(Context.ConnectionId, deviceGroup);
        await Clients.Group(deviceGroup).SendAsync("Notify", $"Client {Context.ConnectionId} has left the group {deviceGroup}");
    }

    // 客户端连接时触发
    public override async Task OnConnectedAsync()
    {
        Console.WriteLine("客户端连接");
    }
}
```
2. 需要在App中指定访问的Url路径
```csharp
//DJ:指定SignalR的访问路径
app.MapHub<ThirdDeviceHub>("SignalR/ThirdDeviceHub");
//DJ：注意需要在app.MpaControllers之前定义访问路径,且需要设置跨域的功能
```
3. 需要注意跨域相关的问题
```csharp
/*DJ:跨域相关设置*/
services.AddCors(options =>
    {
        //更好的在Program.cs中用绑定方式读取配置的方法：https://github.com/dotnet/aspnetcore/issues/21491
        //不过比较麻烦。
        var corsOpt = configuration.GetSection("Cors").Get<CorsSettings>();
        string[] urls = corsOpt.Origins;
        options.AddDefaultPolicy(builder => builder.WithOrigins(urls)
                .AllowAnyMethod().AllowAnyHeader().AllowCredentials());
    }
);
```
4. 客户端相关配置
```csharp
/*配置相关连接参数以及回调方法*/
string hubUrl = "https://127.0.0.1:7253/SignalR/ThirdDeviceHub";

_hubConnection = new HubConnectionBuilder()
    .WithUrl(hubUrl, options =>
    {
        options.HttpMessageHandlerFactory = _ => new HttpClientHandler
        {
            ServerCertificateCustomValidationCallback = (message, cert, chain, errors) => true
        };
    })
    .ConfigureLogging(logging =>
    {
        logging.SetMinimumLevel(LogLevel.Debug);
    })
    .WithAutomaticReconnect()
    .Build();

_hubConnection.On<Dictionary<string, object>>("ReceiveData", data =>
{
    this.Invoke((Action)(() =>
    {
        richTextBox1.AppendText($"Received Data: {FormatData(data)}\n");
    }));
});


/*启动SignalR连接*/

```

