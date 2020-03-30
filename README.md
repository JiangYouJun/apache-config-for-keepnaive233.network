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

# Restart httpd #

```
systemctl restart httpd
```


