---
name: openid_connect.server_conf
language: 
---
    
```
location = /_jwks_uri {
    internal;
    ...
    proxy_set_header    Content-Length "";           
    proxy_set_header    Accept-Encoding "gzip";
    ...
}

location = /_token {
    internal;
    ...
    proxy_set_header    Content-Type "application/x-www-form-urlencoded";
    proxy_set_header    Accept-Encoding "gzip";
    ...
}
```
