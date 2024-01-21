# Documentation

## See the full documentation on https://documentation-com.gitbook.io/netbox-installation/

# Prerequisites

## System Update

```bash
sudo apt update && sudo apt full-upgrade -y
```

## Git Installation

```bash
sudo apt install git -y
```

## Ansible Installation

```bash
sudo apt install ansible -y
```

# PostgreSQL Installation

## PostgreSQL Instalattion

```bash
sudo apt install -y postgresql
```

## PostgreSQL Configuration

### PostgreSQL Version Check

1. Verify that you have installed PostgreSQL 11 or later.

```bash
psql -V
```

### Database Creation

1. Start by invoking the PostgreSQL shell as the system Postgres user.

```bash
sudo -u postgres psql
```

2. Within the shell, enter the following commands to create the database and user (role), substituting your own value for the password.

```sql
CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'password';
ALTER DATABASE netbox OWNER TO netbox;
```

3. Once complete, enter <mark style="color:red;">`\q`</mark> to exit the PostgreSQL shell.

### Verify Service Status <a href="#verify-service-status" id="verify-service-status"></a>

1. You can verify that authentication works by executing the <mark style="color:red;">`psql`</mark> command and passing the configured username and password. (Replace <mark style="color:red;">`localhost`</mark> with your database server if using a remote database.).

```
$ psql --username netbox --password --host localhost netbox
Password:
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

netbox=> \conninfo
You are connected to database "netbox" as user "netbox" on host "localhost" (address "127.0.0.1") at port "5432".
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
netbox=> \q
```

2. If successful, you will enter a `netbox` prompt. Type <mark style="color:red;">`\conninfo`</mark> to confirm your connection, or type <mark style="color:red;">`\q`</mark> to exit.

# Redis Installation

## Redis Installation <a href="#redis-installation" id="redis-installation"></a>

```bash
sudo apt install -y redis-server
```

## Redis Version Check <a href="#redis-installation" id="redis-installation"></a>

```bash
redis-server -v
```

### Verify Service Status <a href="#verify-service-status" id="verify-service-status"></a>

```bash
redis-cli ping
```

If successful, you should receive a <mark style="color:red;">`PONG`</mark> response from the server.

# NetBox Installation

## Install System Packages <a href="#install-system-packages" id="install-system-packages"></a>

```bash
sudo apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev
```

## Download NetBox <a href="#download-netbox" id="download-netbox"></a>

1. Create the base directory for the NetBox installation. For this guide, we'll use <mark style="color:red;">`/opt/netbox`</mark>.

```bash
sudo mkdir -p /opt/netbox/
cd /opt/netbox/
```

2. Next, clone the **master** branch of the NetBox GitHub repository into the current directory. (This branch always holds the current stable release.).

```bash
sudo git clone -b master --depth 1 https://github.com/netbox-community/netbox.git .
```

## Netbox Configuration

### Create the NetBox System User <a href="#create-the-netbox-system-user" id="create-the-netbox-system-user"></a>

1. Create a system user account named <mark style="color:red;">`netbox`</mark>. We'll configure the WSGI and HTTP services to run under this account. We'll also assign this user ownership of the media directory. This ensures that NetBox will be able to save uploaded files.

```bash
sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/
sudo chown --recursive netbox /opt/netbox/netbox/reports/
sudo chown --recursive netbox /opt/netbox/netbox/scripts/
```

### Configuration <a href="#configuration" id="configuration"></a>

1. Move into the NetBox configuration directory and make a copy of <mark style="color:red;">`configuration_example.py`</mark> named <mark style="color:red;">`configuration.py`</mark>. This file will hold all of your local configuration parameters.

```bash
cd /opt/netbox/netbox/netbox/
sudo cp configuration_example.py configuration.py
```

2. Open <mark style="color:red;">`configuration.py`</mark> with your preferred editor to begin configuring NetBox. NetBox offers many configuration parameters, but only the following four are required for new installations:

* <mark style="color:red;">`ALLOWED_HOSTS`</mark>
* <mark style="color:red;">`DATABASE`</mark>
* <mark style="color:red;">`REDIS`</mark>
* <mark style="color:red;">`SECRET_KEY`</mark>

3. If you are not yet sure what the <mark style="color:red;">**domain name**</mark> and/or <mark style="color:red;">**IP address**</mark> of the NetBox installation will be, you can set this to a wildcard (asterisk) to allow all host values:

```
ALLOWED_HOSTS = ['*']
DATABASE = {
    'NAME': 'netbox',
    'USER': 'netbox',
    'PASSWORD': 'password',
    'HOST': 'localhost',
}
```

#### SECRET\_KEY <a href="#secret_key" id="secret_key"></a>

1.  This parameter must be assigned a randomly-generated key employed as a salt for hashing and related cryptographic functions. (Note, however, that it is _never_ directly used in the encryption of secret data.) This key must be unique to this installation and is recommended to be at least 50 characters long. It should not be shared outside the local system.

    A simple Python script named <mark style="color:red;">`generate_secret_key.py`</mark> is provided in the parent directory to assist in generating a suitable key.

```bash
python3 ../generate_secret_key.py
```

### Optional Requirements <a href="#optional-requirements" id="optional-requirements"></a>

1. All Python packages required by NetBox are listed in <mark style="color:red;">`requirements.txt`</mark> and will be installed automatically. NetBox also supports some optional packages. If desired, these packages must be listed in <mark style="color:red;">`local_requirements.txt`</mark> within the NetBox root directory.

#### Remote File Storage

1. By default, NetBox will use the local filesystem to store uploaded files. To use a remote filesystem, install the django-storages library and configure your desired storage backend in <mark style="color:red;">`configuration.py`</mark>.

```bash
sudo sh -c "echo 'django-storages' >> /opt/netbox/local_requirements.txt"
```

### Run the Upgrade Script <a href="#run-the-upgrade-script" id="run-the-upgrade-script"></a>

1. Once NetBox has been configured, we're ready to proceed with the actual installation. We'll run the packaged upgrade script <mark style="color:red;">`upgrade.sh`</mark>.

```bash
sudo /opt/netbox/upgrade.sh
```

### Create a Super User <a href="#create-a-super-user" id="create-a-super-user"></a>

1. NetBox does not come with any predefined user accounts. You'll need to create a super user (administrative account) to be able to log into NetBox. First, enter the Python virtual environment created by the upgrade script.

```bash
source /opt/netbox/venv/bin/activate
cd /opt/netbox/netbox
python3 manage.py createsuperuser
```

### Schedule the Housekeeping Task <a href="#schedule-the-housekeeping-task" id="schedule-the-housekeeping-task"></a>

1. NetBox includes a <mark style="color:red;">**housekeeping**</mark> management command that handles some recurring cleanup tasks, such as clearing out old sessions and expired change records.&#x20;

```bash
sudo ln -s /opt/netbox/contrib/netbox-housekeeping.sh /etc/cron.daily/netbox-housekeeping
```

### Test the Application <a href="#test-the-application" id="test-the-application"></a>

1. At this point, we should be able to run NetBox's development server for testing. We can check by starting a development instance locally.

```bash
python3 manage.py runserver 0.0.0.0:8000 --insecure
```

2. If successful, you should see output similar to the following:

```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
August 30, 2021 - 18:02:23
Django version 3.2.6, using settings 'netbox.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

3. Next, connect to the name or IP of the server (as defined in <mark style="color:red;">`ALLOWED_HOSTS`</mark>) on port 8000; for example, [http://127.0.0.1:8000/](http://127.0.0.1:8000/). You should be greeted with the NetBox home page. Try logging in using the username and password specified when creating a superuser.

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

