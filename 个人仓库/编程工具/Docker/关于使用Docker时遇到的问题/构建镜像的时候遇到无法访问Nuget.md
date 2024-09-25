```Docker
# 使用 .NET 6.0 运行时镜像作为基础镜像
FROM mcr.microsoft.com/dotnet/runtime:6.0 AS base
WORKDIR /app

# 使用 .NET 6.0 SDK 镜像作为构建环境
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src

# 复制 NuGet.config 文件到工作目录
COPY ["NuGet.config","NuGet.config"]

# 复制项目文件到 /src/Modbus_Linux 目录
COPY ["./Modbus_Linux.csproj","Modbus_Linux/"]

# 列出当前目录内容，并输出到日志中
RUN echo "Listing /src:" && ls -la /src

# 设置工作目录为 /src/Modbus_Linux
WORKDIR /src/Modbus_Linux

# 恢复依赖项，使用相对路径
RUN dotnet restore "./Modbus_Linux.csproj" --configfile /src/NuGet.config

# 复制所有项目文件到工作目录
COPY . .

# 列出当前目录内容，并输出到日志中
RUN echo "Listing /src/Modbus_Linux after copy:" && ls -la /src/Modbus_Linux

# 构建项目，并将输出放到 /app/build 目录
RUN dotnet build "./Modbus_Linux.csproj" -c Release -o /app/build

# 基于之前的 build 阶段创建一个新的 publish 阶段
FROM build AS publish
RUN dotnet publish "./Modbus_Linux.csproj" -c Release -o /app/publish /p:UseAppHost=false

# 使用基础运行时镜像作为最终镜像
FROM base AS final
WORKDIR /app

# 从 publish 阶段复制发布输出到当前工作目录
COPY --from=publish /app/publish .

# 设置容器启动时执行的命令
ENTRYPOINT ["dotnet", "Modbus_Linux.dll"]

```

无法访问Nuget报错：
```Docker
#15 [build  7/10] RUN dotnet restore "./Modbus_Linux.csproj" --configfile /src/NuGet.config
#15 0.810   Determining projects to restore...
#15 6.583 /src/Modbus_Linux/Modbus_Linux.csproj : error NU1301: Unable to load the service index for source https://api.nuget.org/v3/index.json.
#15 12.10 /src/Modbus_Linux/Modbus_Linux.csproj : error NU1301: Unable to load the service index for source https://api.nuget.org/v3/index.json.
#15 12.17   Failed to restore /src/Modbus_Linux/Modbus_Linux.csproj (in 11.05 sec).
#15 ERROR: process "/bin/sh -c dotnet restore \"./Modbus_Linux.csproj\" --configfile /src/NuGet.config" did not complete successfully: exit code: 1

```
需要在/etc/docker/daemon.json中配置DNS
```shell
{ "registry-mirrors": [ "https://hub-mirror.c.163.com", "https://ustc-edu-cn.mirror.aliyuncs.com", "https://ghcr.io", "https://mirror.baidubce.com" ], "dns": [ "8.8.8.8", "8.8.4.4" ] }
```
