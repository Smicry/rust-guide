# rust 手册

## 开发环境
推荐在Linux下。

## rust开发环境搭建
### rust组件安装
* rustup component add rust-src
* rustup component add rustfmt
* rustup component add clippy
* rustup component add rust-analysis
* rustup component add rls
### rls2.0(需要自己编译)
* [rust-analyzer](https://github.com/rust-analyzer/rust-analyzer)

### rustup服务器代理
* export RUSTUP_DIST_SERVER=http://mirrors.ustc.edu.cn/rust-static
* export RUSTUP_UPDATE_ROOT=http://mirrors.ustc.edu.cn/rust-static/rustup
### cargo镜像代理
* 在你的CARGO_HOME目录下(默认是~/.cargo)，建立`config`文件：
```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```
## IDE
* code-server，使用二进制部署即可。