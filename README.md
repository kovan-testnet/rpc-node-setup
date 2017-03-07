# RPC Node Setup Steps

## Create VMs

First, create Ubuntu Sever 16.04 virtual machines with default settings...

You'll also need to add a firewall rule for inbound 443 (SSL) and TCP listening port 30303 and UDP discovery port 30301.

## Configure DNS

Use `A` name, pointing the subdomain to the IP address of the VM.

## Connect to VM via SSH

For Example

```bash
ssh ubuntu@<vm ip address>
```

## Install & Start Parity

```bash
bash <(curl https://get.parity.io -Lk)
# say 'no' to the netstats client
tmux
parity --chain=kovan --jsonrpc-hosts=all
# exit tmux, [ctrl + b] then [d]
# later, attach again with;
tmux attach -t 0
```

## Get an SSL Cert

https://certbot.eff.org/#ubuntuxenial-nginx
```bash
sudo apt-get install letsencrypt
sudo letsencrypt certonly --standalone -d <subdomain>.<domain>.com
```

## Nginx Configs

```bash
sudo apt-get install nginx
```

Use the default `/etc/nginx/nginx.conf`

Edit `/etc/nginx/sites-enabled/default`

```
server {

  listen 443 ssl default_server;
  listen [::]:443 ssl default_server;

  ssl_certificate      /etc/letsencrypt/live/<subdomain>.<domain>.com/fullchain.pem;
  ssl_certificate_key  /etc/letsencrypt/live/<subdomain>.<domain>.com/privkey.pem;

  server_name _;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Content-Type application/json;
    add_header Access-Control-Allow-Origin "*";
    add_header Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept";

    if ($request_method = 'OPTIONS') {
      add_header Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept";
      add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
      add_header Access-Control-Allow-Origin "*";
      add_header Access-Control-Max-Age 1728000;
      add_header Content-Type 'text/plain charset=UTF-8';
      add_header Content-Length 0;
      return 204;
    }
    proxy_pass http://localhost:8545;
  }

}
```

Start nginx

```bash
sudo service nginx start # or `restart`
```
