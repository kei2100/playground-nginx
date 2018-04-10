### build image for openssl
docker build -t playground-nginx/openssl -f Dockerfile.openssl .

### create CSR for CA

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl req -new -newkey rsa:2048 -keyout /etc/ssl/CA/private/cakey.pem -out /etc/ssl/CA/certs/careq.pem

----
# promptは下記以外はblankでOK
Enter PEM pass phrase:               # 任意のpass phrase入力
Verifying - Enter PEM pass phrase:   # 任意のpass phrase入力
....
Common Name (e.g. server FQDN or YOUR name) []:TestCA  # CommonName入力
```

### self-sign to CSR for CA

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl ca -create_serial -selfsign -days 3650 -batch -extensions v3_ca \
                      -out /etc/ssl/CA/certs/cacert.pem \
                      -in /etc/ssl/CA/certs/careq.pem \
                      -keyfile /etc/ssl/CA/private/cakey.pem
```

### create CSR for server

```
