<!-- toc -->
# Blackbox exporter：探测远程地址（HTTP HTTPS DNS TCP ICMP）

[blackbox_exporter][1] 用来探测目标地址，支持的协议有：HTTP、HTTPS、DNS、TCP、ICMP。

## 配置文件

blackbox 的配置文件由多个 modules 组成，每个 module 代表一个功能。

在当前目录中准备一个配置文件 [blackbox.yml][3]，里面有 http_2xx、http_post_2xx 等模块：

```yaml
modules:
  http_2xx:
    prober: http
    http:
      headers:
        Accept: "*/*"
  http_post_2xx:
    prober: http
    http:
      method: POST
  tcp_connect:
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
```

[example.yml][2] 中列出了支持的功能模块。

## 启动运行

使用 docker 运行，blackbox.yml 位于当前目录，docker 的安装方法见 [docker 使用手册](../docker/index.md)： 

```sh
docker run --rm -d -p 9115:9115 --name blackbox_exporter -v `pwd`:/config prom/blackbox-exporter:v0.15.1 --config.file=/config/blackbox.yml
```

查看更多镜像 [prom/blackbox-exporter][4]。

## 参考

[1]: https://github.com/prometheus/blackbox_exporter "blackbox_exporter"
[2]: https://github.com/prometheus/blackbox_exporter/blob/master/example.yml "example.yml"
[3]: https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml "blackbox.yml"
[4]: https://hub.docker.com/r/prom/blackbox-exporter/tags "prom/blackbox-exporter"
