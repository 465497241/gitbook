```
server { 
      listen 80; 
      server_name fanpei.com;        //域名
      index  index.html index.php;   //默认首页为index.html
      root "E:/phpstudy/WWW/test";  //路径
      autoindex  on;    //是否显示文件列表
      
      location ~ \.php(.*)$  {                 //可以解析php文件
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  PATH_INFO  $fastcgi_path_info;
            fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
            include        fastcgi_params;
        } 
    }
```



