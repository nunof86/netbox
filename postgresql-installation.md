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
