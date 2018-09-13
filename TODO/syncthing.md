+ uname -a 看系统 是32还是64位
+ 下载地址 https://github.com/syncthing/syncthing/releases
+ mac下载64位 syncthing-macosx-amd64-v0.14.47-rc.1
+ ubuntu下载64位 syncthing-linux-amd64-v0.14.47-rc.1.tar.gz
+ 执行syncthing可执行程序

+ 关闭ubuntu的防火墙

```
ufw disable
```
+ 配置vps文件 在 VPS 上部署时需要修改配置文件

```
/root/.config/syncthing/config.xml
将里面的 IP 地址(默认127.0.0.1)修改为你的 IP 就能远程访问了。
<address>127.0.0.1:8384</address>
```



mac 2WLS7LD-MJNUT2B-HCRSUSE-7FWTZVW-TK4XAUU-HLQNRN2-U66IJH3-DUSFTAE

ubuntu HULNC3C-CBQEXLM-UEPS2QH-JCNY2JK-RF6WJQ2-5N7BKNN-325OBRY-BOM6TQ6