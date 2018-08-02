```
docker build -t playground-nginx .
docker run -v $(pwd)/conf.d:/etc/nginx/conf.d/ -p 8000:80 playground-nginx:latest
```
