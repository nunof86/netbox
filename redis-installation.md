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
