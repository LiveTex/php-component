
```bash
wget https://github.com/LiveTex/php-component/archive/1.0.tar.gz
tar -xzf php-component-1.0.tar
cd php-component-1.0
composer install --no-scripts
cd ..
mv php-component-1.0 project_name
```

```nginx
server {
    server_name offline;

    root /path_to/project_name/public;
    index index.php;

    location / {
        index index.php;
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME /path_to/project_name/public$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
