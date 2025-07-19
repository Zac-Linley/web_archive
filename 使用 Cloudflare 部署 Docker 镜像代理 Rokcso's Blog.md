# 使用 Cloudflare 部署 Docker 镜像代理 | Rokcso's Blog
为了解决 Docker 镜像拉取的网络问题，我尝试过使用网上公开分享的镜像源、阿里云个人镜像源、个人镜像仓库等方式，不是经常失效就是操作繁琐，直到我发现了 [CF-Workers-docker.io](https://github.com/cmliu/CF-Workers-docker.io) 这个项目，这是一个基于 Cloudflare Workers 的 Docker 镜像代理工具。它能够中转对 Docker 官方镜像仓库的请求，可以加速访问和解决一些访问限制的问题。并且部署、使用都非常简单。

部署 & 使用
-------

> 本文所有命令仅在 Ubuntu Server 22.04 LTS 64bit 环境下测试通过。

将这个 [GitHub 仓库](https://github.com/cmliu/CF-Workers-docker.io) Fork 到自己的 GitHub，登录 Cloudflare 后点击「Compute (Workers)」选择「Workers & Pages」，点击「Create」选择「Pages」，点击「Import an existing Git repository」处的「Get started」，授权连接到自己的 GitHub 后选择刚刚 Fork 的 CF-Workers-docker.io 仓库，点击「Begin setup」，所有配置保持默认即可，点击「Save and Deploy」。

项目部署完成后 Cloudflare 默认会提供一个域名，可以选择绑定自己的域名（简化记忆成本）使用，但是直接使用 Cloudflare 提供的域名可能更安全 (⊙\_⊙)? 。

注意：部署的代理服务建议**仅供个人使用**，不要公开分享。

使用时只需要在正常 Pull Docker 时在官方镜像名前加上部署后得到的域名和 `/` 即可，比如：

```bash
docker pull xxx.pages.dev/sissbruecker/linkding:latest

```

这里的 `xxx.pages.dev` 就是刚刚在 Cloudflare 部署后得到的域名。

也可以将上面这个域名添加为 Docker 镜像源，编辑 Docker 配置文件 `daemon.json`：

```bash
sudo nano /etc/docker/daemon.json

```

增加镜像源配置：

```json
{
    "registry-mirrors": ["https://xxx.pages.dev"]
}            

```

可以使用以下命令查看确认一下文件内容：

```bash
cat /etc/docker/daemon.json

```

然后重载配置文件并重启 Docker 服务：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker

```

修改下载的 Docker 镜像名
----------------

发现直接在 Pull Docker 时添加域名的方式下载的镜像文件名也会包含域名，对于后续的使用会比较不方便，所以建议修改为官方镜像名。

首先查看本机存在的所有 Docker 镜像：

输出可能为：

```bash
REPOSITORY                           TAG     IMAGE ID      CREATED      SIZE
xxx.pages.dev/sissbruecker/linkding  latest  de0d3e5r79f1  2 weeks ago  489MB

```

使用以下命令来将旧 REPOSITORY 名映射指定为新 REPOSITORY 名：

```bash
docker tag <旧REPOSITORY>:<TAG> <新REPOSITORY>:<新TAG>

```

比如：

```bash
docker tag xxx.pages.dev/sissbruecker/linkding:latest sissbruecker/linkding:latest

```

这个命令会从旧的镜像复制一个新的镜像并重新命名，所以旧的镜像依然存在，如果旧的镜像没有被其他容器依赖可以选择删除干净：

```bash
docker rmi xxx.pages.dev/sissbruecker/linkding:latest

```