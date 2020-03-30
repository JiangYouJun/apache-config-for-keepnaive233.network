# Install modules #

```
yum install mod_ssl openssl
```

`LoadModule ssl_module modules/mod_ssl.so` already included in `/etc/httpd/conf.modules.d/00-ssl.conf`

# Stop httpd #

```
systemctl stop httpd
```

# Install certs #

```
mkdir -p /etc/pki/httpd/private
sudo ~/.acme.sh/acme.sh --issue -d keepnaive233.network --standalone -k ec-256
sudo ~/.acme.sh/acme.sh --renew -d keepnaive233.network --force --ecc
sudo ~/.acme.sh/acme.sh --installcert -d keepnaive233.network --fullchainpath /etc/pki/httpd/server.crt --keypath /etc/pki/httpd/private/server.key --ecc
```

# Configure Certs in Apache #

Edit `/etc/httpd/conf.d/ssl.conf`

Change these following lines

```
SSLCertificateFile /etc/pki/httpd/server.crt
SSLCertificateKeyFile /etc/pki/httpd/private/server.key
```

# Redirect all http to https #

Edit `/etc/httpd/conf.d/ssl.conf`

Add

```
<VirtualHost *:80>
    <IfModule alias_module>
        Redirect permanent / https://keepnaive233.network/
    </IfModule>
</VirtualHost>
```

# Configure the V2Ray Server #

Edit `/etc/v2ray/config.json`

My full configuration was:

```
{
  "inbounds": [
    {
      "port": 10000,
      "listen":"127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "********-****-****-****-************",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/ray/"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}

```

For privacy, I hided the UUID. You may generate yours by [Online UUID Generator](https://www.uuidgenerator.net/)

After configuring, restart V2Ray

```
systemctl restart v2ray
```


# Configure Reverse Proxy to V2Ray in httpd #

Edit `/etc/httpd/conf.d/ssl.conf`

Add

```
ProxyPass "/ray/" "ws://127.0.0.1:10000/ray/"
ProxyAddHeaders Off
ProxyPreserveHost On
RequestHeader append X-Forwarded-For %{REMOTE_ADDR}s
```

in `<VirtualHost>`

# Restart httpd #

```
systemctl restart httpd
```


# Allow httpd to make outbound connections #

When I finished all the operations, it still didn't work. So I checked the log.

```
cat /var/log/httpd/ssl_error_log 
```

And found this

```
[Mon Mar 30 14:27:21.641856 2020] [proxy:error] [pid 27154] (13)Permission denied: AH00957: WS: attempt to connect to 127.0.0.1:10000 (*) failed
[Mon Mar 30 14:27:21.641933 2020] [proxy_wstunnel:error] [pid 27154] [client 183.92.248.220:8105] AH02452: failed to make connection to backend: 127.0.0.1
```

I used this command to allow httpd to make outbound connections

```
/usr/sbin/setsebool -P httpd_can_network_connect 1
```

And solved the problem.
