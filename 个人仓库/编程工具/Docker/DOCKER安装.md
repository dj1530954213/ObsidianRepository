参考连接：https://blog.csdn.net/u013541325/article/details/124761383
```shell
sudo apt-get update
sudo apt-get install \ apt-transport-https \ software-properties-common \ ca-certificates \ curl \ gnupg \ lsb-release
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
```