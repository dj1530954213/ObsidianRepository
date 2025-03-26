GIT使用代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:10808

在GIT中如果提交了错误的commit可以使用如下命令来撤回
```shell
git reset --soft HEAD~1
//通过git log查看HEAD是否指向了上次的提交
```

GIT中回退至上一次提交的节点
```shell
git reset
```


通过nssm注册git自动推送服务
```shell
nssm install GitAutoPush "D:\GIT\Git\code\GitAutoPush\auto_push_for_git\target\release\auto_push_for_git.exe" "-c D:\GIT\Git\code\GitAutoPush\auto_push_for_git\config.toml -d"
```
