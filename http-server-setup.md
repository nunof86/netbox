# HTTP Server Setup

## Obtain an SSL Certificate <a href="#obtain-an-ssl-certificate" id="obtain-an-ssl-certificate"></a>

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/netbox.key \
-out /etc/ssl/certs/netbox.crt
```

### HTTP Server Installation <a href="#http-server-installation" id="http-server-installation"></a>

### Nginx Installation

```bash
sudo apt install -y nginx
```

### Nginx Configuration

1. Once nginx is installed, copy the nginx configuration file provided by NetBox to <mark style="color:red;">`/etc/nginx/sites-available/netbox`</mark>. Be sure to replace <mark style="color:red;">`netbox.example.com`</mark> with the domain name or IP address of your installation. (This should match the value configured for <mark style="color:red;">`ALLOWED_HOSTS`</mark> <mark style="color:red;"></mark><mark style="color:red;">in</mark> <mark style="color:red;"></mark><mark style="color:red;">`configuration.py`</mark>.).

```nginx
server_name your_ip_address;
```

```bash
sudo cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
```

2. Then, delete <mark style="color:red;">`/etc/nginx/sites-enabled/default`</mark> and create a symlink in the <mark style="color:red;">`sites-enabled`</mark> directory to the configuration file you just created.

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox
```

3. Finally, restart the `nginx` service to use the new configuration.

```bash
sudo systemctl restart nginx
```

4. After that navigate to the dashboard with <mark style="color:red;">`https://your_ip_address`</mark> with the credentials (<mark style="color:red;">admin</mark>/<mark style="color:red;">`adminpassword`</mark>).
