# Clash使用指南

## 一、安装Clash

```shell
yay clash
# or sudo pacman -S clash
```



## 二、`config.yml`与`Country.mmdb`

`config.yml`一般是机场会给的一系列节点的相关信息的文件。

`Country.mmdb`是全球IP库，可以实现各个国家的IP信息解析和地理定位，为了以后使用方便，`Country.mmdb`文件就直接放在项目里，不放心的话，Github上就有，链接地址如下：[Country.mmdb 地址](https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb)。

这两个文件是Clash使用必不可少的文件，通常我会根据不同的机场名字在`~/.config/clash`下建立不同的文件夹，每个文件夹里都有这两个文件。

此外，一般我们会在`~/.bashrc`中添加如下内容：

```
export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"
export all_proxy="http://127.0.0.1:7890"
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"
export ALL_PROXY="http://127.0.0.1:7890"
```

因为一般机场的Clash配置yml文件中会设置HTTP代理端口为7890，SOCKS端口为7891。

要使用对某家机场的节点，比方说`N3RO`，`clash -d ~/.config/clash/N3RO`，clash就跑起来了。



###  三、Linux下Clash的Web UI—Clash Dashboard

在Clash运行起来之后，通过浏览器访问`http://clash.razord.top/`，就可以进入Clash Dashboard，在这里可以选择代理，进行Clash的相关设置。

> 如果使用的机场不会进行在线Clash节点更新的话，最好隔几天就用新的yml文件覆盖旧的yml文件。

一般我会首先吧Clash的代理模式设置为全局的，然后在Chrome里下载Proxy SwitchyOmega，选择用clash的模式，之后将Clash切换回代理模式为规则，当然一直使用全局的可以，正常的机场一个月的流量足够用了。