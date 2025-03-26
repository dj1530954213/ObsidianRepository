clea**创建镜像**：
```shell
docker build --no-cache --progress=plain -t auth .
```
**运行镜像**: 
```shell
docker run -d -p 5502:5502 --name my-dotnet-app-container my-dotnet-app
#-d：让容器以分离（detached）模式运行，即在后台运行。
#-p 5502:5502`：将主机的端口`5502`映射到容器的端口`5502`。
#--name my-dotnet-app-container`：为容器指定一个名称，便于管理和识别。
#my-dotnet-app`：这是你的镜像名称。
```
**查看运行的容器**:
```shell
docker ps
```
**查看容器日志**:
```shell
docker logs my-dotnet-app-container
```

**停止容器**
```shell
docker stop my-dotnet-app-container
```
**删除容器**
```shell
docker rm my-dotnet-app-container
```

