---
title: 设置 Docker 代理
tag: devops
---

# 设置 docker 代理

* 设置代理:

  * 创建 `docker.service.d` 目录
    ```
    $ mkdir -p /etc/systemd/system/docker.service.d
    ```
    
  * `HTTP_PROXY`:
    ```
    $ cat <<EOF > /etc/systemd/system/docker.service.d/http-proxy.conf
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80/"
    EOF
    ```
    
  * `HTTPS_PROXY`:
    ```
    $ cat <<EOF > /etc/systemd/system/docker.service.d/https-proxy.conf
    [Service]
    Environment="HTTPS_PROXY=https://proxy.example.com:443/"
    EOF
    ```
    
* 排除代理:
  * `HTTP_PROXY`:
    ```
    Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
    ```
  * `HTTPS_PROXY`:
    ```
    Environment="HTTPS_PROXY=https://proxy.example.com:443/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
    ```
* 重启 `docker`
    ```
    $ systemctl daemon-reload
    $ systemctl restart docker
    ```
    
* 检查是否生效
  * `HTTP_PROXY`:
    ```
    $ systemctl show --property=Environment docker
    Environment=HTTP_PROXY=http://proxy.example.com:80/
    ```
  
  * `HTTPS_PROXY`:
    ```
    $ systemctl show --property=Environment docker
    Environment=HTTPS_PROXY=https://proxy.example.com:443/
    ```
