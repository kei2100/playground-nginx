## Prepare
### build image for openssl
docker build -t playground-nginx/openssl -f Dockerfile.openssl .

## CA
### create CSR for CA
```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl req -new -keyout ./CA/private/cakey.pem -out ./CA/careq.pem
```

### self-sign to CSR for CA
```bash
touch ./openssl-etc/ssl/CA/db/index.txt

docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl ca -selfsign -create_serial -in ./CA/careq.pem -out ./CA/cacert.pem -extensions ca_ext
```

## Server
### create CSR for Server
TLSするためにServer証明書を作成しておく。

```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl req -new -keyout ./server/private/servkey.pem -out ./server/servreq.pem
```

### sign to CSR for Server
```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl ca -days 825 -in ./server/servreq.pem -out ./server/servcert.pem -extensions server_ext
```

### generate NO-PASS-PHRASE key for Server
```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl rsa -in ./server/private/servkey.pem -out ./server/private/servkey-nopass.pem
```

## Client
### create CSR for Client
```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl req -new -keyout ./client/private/clikey.pem -out ./client/clireq.pem
```

### sign to CSR for Client
```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl ca -days 825 -in ./client/clireq.pem -out ./client/clicert.pem -extensions client_ext
```

### change Client cert format to PKCS#12
```bash
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
  openssl pkcs12 -export -in ./client/clicert.pem -inkey ./client/private/clikey.pem -out ./client/clicert.pfx
```

OSXであればclicert.pfxを開くなどしてキーチェーンにインポートする

## Revoke certificate
### create or update CRL list
```
# crlnumberがなければ作成しておく
echo '00' > openssl-etc/ssl/CA/db/crlnumber

docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl ca -updatedb -gencrl -out ./CA/crl/crl.pem 
```

### revoke
```
# ./CA/newcertsから探しても良い
docker run -it -v $(pwd)/openssl-etc/ssl:/etc/ssl playground-nginx/openssl:latest \
 openssl ca -revoke ./client/revoke_test_cert.pem
```

## Test
### run Nginx
```bash
 docker run -p 8000:80 -p 443:443 -v $(pwd)/openssl-etc/ssl:/etc/ssl -v $(pwd)/nginx-etc/nginx/conf.d:/etc/nginx/conf.d  playground-nginx:latest
```

https://localhost/ からのレスポンスヘッダ `X-SSL-Client-Verify` にクライアント認証の結果が入る
