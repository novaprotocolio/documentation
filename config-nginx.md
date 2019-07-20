# Config nginx for your dex external accesses

## Why need nginx?

We can simply export port on `0.0.0.0` to make our servie accessable on internet. But it's not safe and efficient.

By using nginx, you can:

- Serve static files, which make your web access faster.
- Only reverse proxy necessary services and keep the other services in `localhost`, which makes your dex safe.
- Easy to support ssl, which makes your dex safer.
- Access control
- Load balancing

## Prepare

We assume that you can successfully run a layerer, and you can access the webpage through the `localhost:3000` port, which works fine. If not, please follow the [README](/#quickstart) and try to launch a dex in localhost first.

## Step 1: Install nginx

Please follow the official [document](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/) to install.

## Step 2: Configure your services

Firstly, you need some domains for external accesses. Let's make a example here. You should replace the corresponding domain to yours.

- `dex-demo.site.io` for web service
- `dex-demo-api.site.io` for api service
- `dex-demo-ws.site.io` for websocket service
- `dex-demo-rpc.site.io` for ethereum test rpc service

Update your `web` service environment variables in your `docker-compose.yaml`.

```diff
-      REACT_APP_API_URL=http://localhost:3001
+      REACT_APP_API_URL=http://dex-demo-api.site.io

-      REACT_APP_WS_URL=ws://localhost:3002
+      REACT_APP_WS_URL=ws://dex-demo-ws.site.io

// If you are using the Ethereum test rpc server, you should also change:

-      REACT_APP_NODE_URL=http://localhost:8545
+      REACT_APP_NODE_URL=http://dex-demo-rpc.site.io
```

Restart your web service if it is already running. You can use `docker-compose restart web` command.

The following is a nginx config template for nova dex. Edit it appropriately, then put the config into the `http` section in nginx config.

```nginx
upstream nova-dex-web {
    server localhost:3000;
}

upstream nova-dex-api {
    server localhost:3001;
}

upstream nova-dex-ws {
    server localhost:3002;
}

# If you are running a localhost ethereum node, you need to uncomment the following upstream
# upstream nova-dex-rpc {
#     server localhost:8545;
# }

proxy_set_header Host              $http_host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

server {
    listen 80;
    server_name dex-demo.site.io;

    # cache static content
    location ^~ /static/ {
        expires    7d;
        proxy_pass http://nova-dex-web;
    }

    location / {
        proxy_pass http://nova-dex-web;
    }
}

server {
    listen 80;
    server_name dex-demo-api.site.io;
    location / {
        proxy_pass http://nova-dex-api;
    }
}

server {
    listen 80;
    server_name dex-demo-ws.site.io;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://nova-dex-ws;
    }
}

# If you are running a localhost ethereum node, you need to uncomment the following server
# server {
#     listen 80;
#     server_name dex-demo-rpc.site.io;
#     location / {
#         proxy_pass http://nova-dex-rpc;
#     }
# }

```

**SSL**: You can add ssl for your nginx to support `https` and `wss` accesses. You can buy a ssl certification or use free ssl certification issued by [Let's encrypt](https://letsencrypt.org/). This will be some addition effort on nginx config. [Cloudflare](https://cloudflare.com) free ssl encrypt is also a good choice.

**Note**: If you are using an aws EC2 server, make sure your EC2 security group allows TCP traffic on ports 80 and 443. If you run the test ethereum node, you also need to allow TCP traffic on port 8545 for web wallet access.

## Step 3: Reload nginx

Firstly, you should test your nginx config with `sudo nginx -t` command. It will either show error messages when something goes wrong, or succeed without print anything. Then, use `sudo nginx -s reload` command to reload your nginx.

## Step 4: Try

Now, you can access your dex with your host name. In this example, open `http://dex-demo.site.io` url in your browser.
