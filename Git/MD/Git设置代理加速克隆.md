# Git设置代理加速克隆

**注意**：一定要有代理，且开启并可用。

配置全局代理，在git bash中输入：

```bash
git config --global http.proxy socks5://127.0.0.1:10808
git config --global https.proxy socks5://127.0.0.1:10808
```

端口根据自己代理的配置填写，v2ray的一般是10808。

取消代理修改：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

全局代理，但是如果要克隆码云、coding等国内仓库，速度就会很慢。

更好的方法是只对github进行代理，不会影响国内仓库：

```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:10808
git config --global https.https://github.com.proxy socks5://127.0.0.1:10808
```
