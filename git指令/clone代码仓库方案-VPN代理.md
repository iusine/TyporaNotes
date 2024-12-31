### GitHub仓库代码拉取

> 在使用 Git 进行代码管理时，可能会遇到“Failed to connect to github.com port 443 after 21090 ms: Couldn’t connect to server”这种连接失败的错误提示。这个问题常常与网络配置、代理设置或 VPN 环境的干扰有关。
>

#### 1. 检查当前代理设置

首先，确认你系统的代理设置。通常，VPN 会配置一个本地代理端口来进行网络请求。你可以通过以下步骤检查代理端口：

1. 打开 **设置** > **网络与互联网** > **代理**，找到代理编辑，点击编辑,并记录当前代理端口。

<img src="https://jsd.onmicrosoft.cn/gh/iusine/TyporaNotesPicture/image-20241229213630146.png" alt="image-20241229213630146" style="zoom:80%;" />

#### 2. 配置 Git 使用代理

确保 Git 使用与系统代理设置相同的端口。可以通过以下命令配置 Git 的代理：

以我的端口1314为例

> git config --global http.proxy http://127.0.0.1:1314
> git config --global https.proxy http://127.0.0.1:1314

#### 3.验证代理设置是否生效

在配置完成后，你可以使用以下命令验证代理设置是否正确：

```
git config --global -l
```


这将列出当前的 Git 配置信息，确保其中的 http.proxy 和 https.proxy 设置为你刚刚配置的端口。

#### 4.刷新 DNS 缓存

有时 DNS 缓存可能会导致连接问题。在执行 Git 操作前，建议刷新系统的 DNS 缓存：

**Windows 用户：**

```
ipconfig /flushdns
```

**Mac 用户：**

```
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

#### 5.重新尝试 Git 操作

在完成上述步骤后，尝试执行 git push 或 git pull 等 Git 命令，看看是否能成功连接并操作 GitHub。如果问题仍然存在，请检查网络连接是否稳定，或者尝试更换 VPN 服务器。