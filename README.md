# mattermost-formula

This sets up Mattermost, an open source alternative to Slack.

See the full [Salt Formulas installation and usage instructions](http://docs.saltstack.com/en/latest/topics/development/conventions/formulas.html).

## Available states

``mattermost``
------------

Installs the mattermost archive, and starts the associated mattermost service.


``mattermost.uninstall``
------------------

Uninstalls the mattermost archive, and removes the associated mattermost service.


## webserver and database

Feel free to use your favorites. I prefer the `postgres-formula` together with
nginx vhost template files in local states.

### nginx server sample

This one uses nginx with 80 to 443 redirect and proxy configuration.

__NOTE__: The websocket endpoint needs additional tweaks to work properly.

```nginx
server {

  listen 80;
  server_name your.domain;

  location / {
    rewrite ^ https://$host$request_uri;
  }
}

server {
  listen 443 ssl;
  server_name your.domain;
  client_max_body_size 10M;
  ssl_certificate /path/to/fullchain.pem;
  ssl_certificate_key /path/to/key.pem;

  location = /favicon.ico {
    log_not_found off;
    access_log off;
  }

  location / {
    proxy_pass http://localhost:8065;
    client_max_body_size 128m;
    client_body_buffer_size 128k;

    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
  	proxy_connect_timeout 240;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Connection "";

    proxy_redirect off;
  	proxy_http_version 1.1;
    proxy_cache_bypass $cookie_session;
  	proxy_no_cache $cookie_session;
  	proxy_buffers 32 4k;
  }

  location /api/v3/users/websocket {
    proxy_pass http://localhost:8065/api/v3/users/websocket;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```
