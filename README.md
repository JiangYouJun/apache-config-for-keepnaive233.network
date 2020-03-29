# Install modules #

```
yum install mod_ssl openssl
```

`LoadModule ssl_module modules/mod_ssl.so` already included in `/etc/httpd/conf.modules.d/00-ssl.conf`

# Install certs #

```
systemctl stop httpd
mkdir -p /etc/pki/httpd/private
sudo ~/.acme.sh/acme.sh --issue -d keepnaive233.network --standalone -k ec-256
sudo ~/.acme.sh/acme.sh --renew -d keepnaive233.network --force --ecc
sudo ~/.acme.sh/acme.sh --installcert -d keepnaive233.network --fullchainpath /etc/pki/httpd/server.crt --keypath /etc/pki/nginx/private/server.key --ecc
```

# Edit /etc/httpd/conf.d/ssl.conf #

```
SSLCertificateFile /etc/pki/httpd/server.crt
SSLCertificateKeyFile /etc/pki/nginx/private/server.key
```

# Restart httpd #

```
systemctl restart httpd
```


