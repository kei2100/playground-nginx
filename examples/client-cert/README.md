### build image for openssl
docker build -t playground-nginx/openssl -f Dockerfile.openssl .

### create CSR for CA

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl req -new -newkey rsa:2048 -keyout ./CA/private/cakey.pem -out ./CA/careq.pem

----
# promptは下記以外はblankでOK
Enter PEM pass phrase:               # 任意のpass phrase入力
Verifying - Enter PEM pass phrase:   # 任意のpass phrase入力
....
Common Name (e.g. server FQDN or YOUR name) []:TestCA  # CommonName入力
```

### self-sign to CSR for CA

```bash
touch ./openssl-etc/ssl/CA/index.txt

docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl ca -create_serial -selfsign -days 1095 -batch -extensions v3_ca \
                      -out ./CA/cacert.pem \
                      -in ./CA/careq.pem \
                      -keyfile ./CA/private/cakey.pem
```

### create CSR for Server

TLSするためにServer証明書を作成しておく。

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl req -new -newkey rsa:2048 -keyout ./server/private/servkey.pem -out ./server/servreq.pem
```

### sign to CSR for Server

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl ca -policy policy_anything -days 825 -out ./server/servcert.pem -infiles ./server/servreq.pem
```

### generate NO-PASS-PHRASE key for Server

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl rsa -in ./server/private/servkey.pem -out ./server/private/servkey-nopass.pem
```
