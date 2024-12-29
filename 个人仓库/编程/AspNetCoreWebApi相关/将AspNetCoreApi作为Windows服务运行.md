第一步：创建WebApi程序
第二步：将项目输出类型由控制台应用程序改为Windows应用程序
![[Pasted image 20241118170252.png]]
第三步：通过NUGET安装第三方库Microsoft.Extensions.Hosting.WindowsServices（注意版本）
第四部：在Program中添加UseWindowsService的配置
```csharp
public static void Main(string[] args)
{
    // 初始化 NLog 配置
    var logger = LogManager.Setup()
        .LoadConfigurationFromFile("NLog.config")
        .GetCurrentClassLogger();

    try
    {
        // 设置当前目录为程序路径
        Directory.SetCurrentDirectory(AppContext.BaseDirectory);

        // 用于创建日志文件夹
        Directory.CreateDirectory(Path.Combine(AppContext.BaseDirectory, "log"));

        var builder = WebApplication.CreateBuilder(args);

        // 清除所有日志提供程序并仅使用 NLog
        builder.Logging.ClearProviders(); // 清除默认日志提供程序
        builder.Logging.SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Debug); // 设置最低日志级别
        builder.Host.UseNLog(); // 启用 NLog

        // 配置应用作为 Windows 服务运行（无 EventLog 集成）
        builder.Host.UseWindowsService();

        // Add services to the container
        builder.Services.AddControllers();
        builder.Services.AddEndpointsApiExplorer();
        //api页面相关设置
        builder.Services.AddSwaggerGen(c =>
        {
            c.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo { Title = "My API", Version = "v1" });
        });
        //配置跨域端口可以使用指定也可是允许所有
        //builder.Services.AddCors(options =>
        //    {
        //        options.AddDefaultPolicy(builder => builder.WithOrigins("http://localhost:4200")
        //            .AllowAnyMethod().AllowAnyHeader().AllowCredentials());
        //    }
        //);
        builder.Services.AddCors(options =>
        {
            options.AddDefaultPolicy(policy =>
            {
                policy.AllowAnyOrigin()
                    .AllowAnyMethod()
                    .AllowAnyHeader();
            });
        });

        // 显式绑定监听端口,否则将在默认端口上监听
        builder.WebHost.UseUrls("http://localhost:5021");
        // 加载配置文件
        builder.Configuration.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true);
        builder.Configuration.AddEnvironmentVariables();
        var app = builder.Build();

        // 设置API文档显示相关配置
        app.UseSwagger();
        app.UseSwaggerUI(c =>
        {
            c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
            c.RoutePrefix = string.Empty;
        });
        //设置跨域
        app.UseCors();

        app.UseHttpsRedirection();
        app.UseAuthorization();
        app.MapControllers();

        app.Run();
    }
    catch (Exception ex)
    {
        // 捕获异常并记录
        logger.Error(ex, "Stopped program because of an exception");
        throw;
    }
    finally
    {
        // 确保日志资源释放
        LogManager.Shutdown();
    }
}
```
第五步：发布程序到文件夹
第六步：创建windows服务
```shell
sc create "MyAspNetCoreService" binPath= "C:\Path\To\YourApp\YourApp.exe"


例如：
sc create "TestAspNet" binPath= "C:\Users\DJ\Desktop\北栈桥联锁控制软件\BZQ_LOCK.exe"
```
第七步：在服务管理中运行
