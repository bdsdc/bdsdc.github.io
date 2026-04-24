# 用nginx 和caddy 分别代理pve，忽略pve证书

## 需求
本身pve 控制台需要开启证书，但是证书是不安全的，也不是合法证书，合法证书还需要通过acme 或者其他工具来实现
所以想先通过nginx 代理pve ，然后向上游访问pve访问的时候，忽略pve证书，要不然可能会跳转失败

## nginx代理
```shell
server {
    listen                              80;
    listen                              443 ssl;
    server_name                         pve-zero.bdser.cc;
    include                             ssl_bdser.cc.conf;

    access_log                          /data/logs/pve-zero.ecarxmap.com-access.log json;

    if ($scheme = http ) {
        return 301 https://$host$request_uri;
    }

    if ( $front_real_ip !~* "^(192)\.(|168)\.([\d]{1,3})\.([\d]{1,3})$" ) {
        return 403;
    }

    location / {
        proxy_pass                      https://192.168.31.100:8006;
        # 上传文件大小控制
        client_max_body_size 15g;
        # 忽略上游https证书验证
        proxy_ssl_verify off;
        proxy_ssl_session_reuse off;
        # websocket （pve 控制台）
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade"; 
        # 超时配置
        proxy_connect_timeout 120s;
        proxy_send_timeout 120s;
        proxy_read_timeout 120s;
        # 标准配置 
        proxy_set_header                Host $host;
        proxy_set_header                X-Forwarded-For $front_real_ip;

    }
}


```
### caddy 配置

正常都可以访问
```shell
pve.yourdomain.com {
    reverse_proxy https://pve-server-ip:8006 {
        transport http {
            tls_insecure_skip_verify
        }
    }
}


```

加上访问控制 

```shell
pve.yourdomain.com {
    # 只允许内网访问
    @internal {
        remote_ip 192.168.0.0/16 10.0.0.0/8 127.0.0.1
    }

    # 符合内网条件 → 反代 PVE
    handle @internal {
        reverse_proxy https://192.168.1.100:8006 {
            transport http {
                tls_insecure_skip_verify  # 忽略PVE自签名证书
            }
        }
    }

    # 否则拒绝
    handle {
        respond 403
    }
}

```



