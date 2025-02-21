## Set up the repository
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```
## Add Docker’s official GPG key
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

```

## Use the following command to set up the repository
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```
## install
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo update-alternatives --config iptables
```
参考：[https://blog.csdn.net/qq_55621259/article/details/134278500](https://blog.csdn.net/qq_55621259/article/details/134278500)

## 启动docker
```
sudo service docker start
sudo service docker status
sudo docker run hello-world
```
如果拉不下来镜像，尝试用这个命令
```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://hub.rat.dev",
    "https://docker.1panel.live"
  ]
}
EOF
```
然后重启docker服务就可以 sudo service docker start or stop