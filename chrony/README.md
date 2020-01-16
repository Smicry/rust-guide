# chrony
* chrony是一个ntp协议的实现程序，既可以当做服务端，也可以充当客户端；
* 它专为间歇性互联网连接的系统而设计，当然也能良好应用于持久互联网连接的环境；

## chrony有三个时间参考
* 硬件时钟
* 实时时钟
* 手动同步

## 程序环境
* 可以通过"man chrony.conf"了解主配置文件的配置格式。
```
# rpm -ql chrony
...
/etc/chrony.conf      // 主配置文件
/usr/bin/chronyc      // 客户端程序
/usr/sbin/chronyd     // 服务端程序
...
```
