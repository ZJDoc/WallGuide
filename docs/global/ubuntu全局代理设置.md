
# Ubuntu全局代理设置

配置[SwitchyOmega代理设置](../plugin/switchy-omega.md)仅仅能够实现浏览器代理，还需要设置全局代理才能在命令行访问国外资源

## 生成pac文件

安装`pengac`后生成`pac`文件

```
$ genpac --proxy="SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
```

## 系统配置

打开系统设置`->Network->Network proxy`，选择`Method`为`Automatic`，在`Configuration URL`中填入生成的文件路径

```
file://文件路径
# 我的
file:///home/zj/software/vpn/autoproxy.pac
```

点击`Apply system wide`即可生效

## 代理工具设置

代理操作使用`SOCKS5`协议，大多命令行操作使用`HTTP/HTTPS`协议，所以需要使用代理工具进行转换。可以使用工具 `proxychains` 或者 `polipo`，以 `proxychains` 为例

### 安装ProxyChains

安装：

    sudo apt-get install proxychains

修改配置文件 `/etc/proxychains.conf`

    #socks4 	127.0.0.1 9050
    socks5 127.0.0.1 1080

重启应用
    
    sudo /etc/init.d/proxychains restart

### 测试命令

```
$ proxychains curl www.google.com
# 或
$ proxychains wget www.google.com
```

### 问题一：proxychains ping www.google.com出错
    
    ProxyChains-3.1 (http://proxychains.sf.net)
    ERROR: ld.so: object 'libproxychains.so.3' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored.
    PING www.google.com (74.86.17.48) 56(84) bytes of data.

这里面共有两个问题，第一个问题：没有预加载 `libproxychains.so.3`

查找：

    locate libproxychains.so.3

    /usr/lib/x86_64-linux-gnu/libproxychains.so.3
    /usr/lib/x86_64-linux-gnu/libproxychains.so.3.0.0

可以修改预加载文件 `/usr/bin/proxychains`

    cat /usr/bin/proxychains

    #!/bin/sh
    echo "ProxyChains-3.1 (http://proxychains.sf.net)"
    if [ $# = 0 ] ; then
    	echo "	usage:"
    	echo "		proxychains <prog> [args]"
    	exit
    fi
    export LD_PRELOAD=libproxychains.so.3
    exec "$@"

或者自己添加到环境变量

    export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libproxychains.so.3

### 问题二：proxychains 对 ping 没用

`proxychains` 使用的协议和 `ping` 使用的协议不同

## 相关阅读

* [Ubuntu代理配置](./Ubuntu代理配置.md)
* [ubuntu使用shadowsocks设置全局代理](./Ubuntu代理配置.md)
* [proxychains-ld-preload-cannot-be-preloaded](https://askubuntu.com/questions/293649/proxychains-ld-preload-cannot-be-preloaded)
* [Is ping not supposed to work via proxychains?](https://superuser.com/questions/442995/is-ping-not-supposed-to-work-via-proxychains)
* [Is ping not supposed to work via proxychains?](https://superuser.com/questions/442995/is-ping-not-supposed-to-work-via-proxychains)