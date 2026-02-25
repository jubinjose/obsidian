user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    # --- Custom Log Formats ---
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # New format to track backend performance
    log_format  backend_log '$remote_addr - $remote_user [$time_local] '
                            '"$request" $status $body_bytes_sent '
                            'to: $upstream_addr | upstream_status: $upstream_status '
                            'rt: $request_time urt: $upstream_response_time';

    # Use the new format for the main access log
    access_log  /var/log/nginx/access.log  backend_log;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # --- Gzip Compression ---
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 256;
    gzip_types
        text/plain
        text/css
        application/json
        application/javascript
        application/x-javascript
        text/xml
        application/xml
        application/xml+rss
        text/javascript
        image/svg+xml;

    include /etc/nginx/conf.d/*.conf;

    # --- HTTP: Redirect all to HTTPS ---
    server {
        listen       80;
        server_name  test.scrambleid.com test1.scrambleid.com;
        return 301 https://$host$request_uri;
    }

    # --- HTTPS: Main Server Block ---
    server {
        listen       443 ssl http2 default_server;
        server_name  test.scrambleid.com test1.scrambleid.com;
        root         /usr/share/nginx/html;

        # SSL Configuration (Certbot Managed)
        ssl_certificate /etc/letsencrypt/live/test.scrambleid.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/test.scrambleid.com/privkey.pem;
        
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Security Headers
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        # Global Proxy Settings
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        include /etc/nginx/default.d/*.conf;

        # --- Locations ---
        location / {
            proxy_pass http://127.0.0.1:5001/;
        }

        location /sp/ {
            proxy_pass http://127.0.0.1:5001/;
        }

        location /helper/ {
            proxy_pass http://127.0.0.1:7001/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }

        location /oidc-server/ {
            proxy_pass http://127.0.0.1:5002/;
        }

        location /echo/ {
            proxy_pass http://127.0.0.1:5003/;
        }

        location /b2bclient/ {
            proxy_pass http://127.0.0.1:5004/;
        }

        location /b2b/ {
            proxy_pass http://127.0.0.1:3000/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            sub_filter_once off;
            sub_filter '/assets/' '/b2b/assets/';
        }
	
	location /beehive/ {
	    proxy_pass http://127.0.0.1:9001;
	    proxy_http_version 1.1;
	    proxy_set_header Upgrade $http_upgrade;
	    proxy_set_header Connection "upgrade";
	    proxy_set_header Host $host;
	    proxy_set_header X-Forwarded-Proto $scheme;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}

	location = /beehive {
	    proxy_pass http://127.0.0.1:9001/beehive;
	    proxy_http_version 1.1;
	    proxy_set_header Host $host;
	    proxy_set_header X-Forwarded-Proto $scheme;
	}

        
        # Error Pages
        error_page 404 /404.html;
        location = /40x.html {}

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {}
    }
}
