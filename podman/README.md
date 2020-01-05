# [podman](https://github.com/containers/libpod)

## docker区别
* `Docker CLI` 会通过 `gRPC API` 去跟 `Docker Engine` 说「我要启动一个容器」，然后 `Docker Engine` 才会通过 `OCI Container runtime`（默认是 `runc` ）来启动一个容器。这就意味着容器的进程不可能是 Docker CLI 的子进程，而是 Docker Engine 的子进程。
* `Podman` 不使用 `Daemon`，而是直接通过 `OCI runtime`（默认也是 `runc` ）来启动容器，所以容器的进程是 `podman` 的子进程。这比较像 `Linux` 的 `fork/exec` 模型，而 `Docker` 采用的是 `C/S`（客户端/服务器）模型。与 `C/S` 模型相比，`fork/exec` 模型有很多优势，比如：
    * 系统管理员可以知道某个容器进程到底是谁启动的。
    * 如果利用 cgroup 对 podman 做一些限制，那么所有创建的容器都会被限制。
    * SD_NOTIFY : 如果将 podman 命令放入 systemd 单元文件中，容器进程可以通过 podman 返回通知，表明服务已准备好接收任务。
    * socket 激活 : 可以将连接的 socket 从 systemd 传递到 podman，并传递到容器进程以便使用它们。
    
## 替代docke
* `bash` 中加入 `alias docker=podman` 完成替换
## 

## 设置自启动
* `Systemd` 来实现 `Podman` 开机重启容器，建立一个 Systemd 服务配置文件：
```
$ vim /etc/systemd/system/nginx_podman.service

[Unit]
Description=Podman Nginx Service
After=network.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/podman start -a nginx
ExecStop=/usr/bin/podman stop -t 10 nginx
Restart=always

[Install]
WantedBy=multi-user.target
```
* 接下来，启用这个 Systemd 服务
```
$ sudo systemctl daemon-reload
$ sudo systemctl enable nginx_podman.service
$ sudo systemctl start nginx_podman.service
```