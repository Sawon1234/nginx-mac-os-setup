Setting up NGINX server on Mac OS X El Kapitan

1. Assuming that you have Homebrew installed, run the following command to install NGINX
   brew install nginx
   
2. Should you run into any issues with respect to any permissions or missing libraries, use the following 
   shell script and run it once
   ```
   #!/bin/sh
   #reinstall everything you currently have in brew 
   for f in `brew list`; do
      echo $f
      brew uninstall $f &&  brew install $f
   done
   ```
3. Use the following command to stop and start the NGINX server
   sudo nginx -s stop && sudo nginx
   
4. Now assume that you have a UI application (AngularJS app) that renders only static content and is backed by a REST API which could be
   either running on the same machine or a different host, you would probably need to route the requests to the REST API. The
   following nginx.conf shows one such reverse proxy routing to a REST API that runs on port 9000
   
 ```  
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8080;
        server_name  localhost;
        root /Users/joesan/Projects/Sandbox/my-app;

        #charset koi8-r;

        access_log  logs/host.access.log  main;

        location / {
                # If you want to enable html5Mode(true) in your angularjs app for pretty URL
                # then all request for your angularJS app will be through index.html
                try_files $uri /index.html;
        }

        # /api will server your proxied API that is running on same machine different port
        # or another machine. So you can protect your API endpoint not get hit by public directly
        location /api {
                proxy_pass http://localhost:9000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```

So effectively any request to /api will be reouted to the REST API backend server and any request coming in at port 8080 
will be served by the static AngularJS application.
