# Gunicorn Configuration

## Configuration

1. NetBox ships with a default configuration file for gunicorn. To use it, copy <mark style="color:red;">`/opt/netbox/contrib/gunicorn.py`</mark> to <mark style="color:red;">`/opt/netbox/gunicorn.py`</mark>.&#x20;

```bash
sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
```

## Systemd Setup <a href="#systemd-setup" id="systemd-setup"></a>

1. We'll use systemd to control both gunicorn and NetBox's background worker process. First, copy <mark style="color:red;">`contrib/netbox.service`</mark> and <mark style="color:red;">`contrib/netbox-rq.service`</mark> to the <mark style="color:red;">`/etc/systemd/system/`</mark> directory and reload the systemd daemon.

```bash
sudo cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
sudo systemctl daemon-reload
```

2. Then, start the <mark style="color:red;">`netbox`</mark> and <mark style="color:red;">`netbox-rq`</mark> services and enable them to initiate at boot time.

```bash
sudo systemctl start netbox netbox-rq
sudo systemctl enable netbox netbox-rq
```
