# ca-certificates [![](https://images.microbadger.com/badges/version/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own version badge on microbadger.com")  [![](https://images.microbadger.com/badges/image/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/commit/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own commit badge on microbadger.com")
Create CA and certificates based on it.

## Execution:

### Create CA, first execution:
```shell
docker run --rm \
  -e SSL_SUBJECT=andradaprieto.es \
  -e SSL_DNS=andradaprieto \
  -v $(pwd)/certs:/certs \
  jorgeandrada/ca-certificates:latest
```

### Creation of certificates, subsequent executions:
```shell
docker run --rm \
  -e SSL_SUBJECT=blog.andradaprieto.es \
  -e SSL_DNS=blog \
  -v $(pwd)/certs:/certs \
  jorgeandrada/ca-certificates:latest
```
### Generate Wildcard cert
```shell
docker run --rm \
  -e SSL_SUBJECT="*.andradaprieto.es" \
  -e SSL_DNS="*" \
  -v $(pwd)/certs:/certs \
  jorgeandrada/ca-certificates:latest
```

## Advanced Usage

Customize the certs using the following Environment Variables:
* `DEBUG` debug level 0/1/2, default `0`
* `CA_KEY` CA Key file, default `ca-key.pem` __[1]__
* `CA_CERT` CA Certificate file, default `ca.pem` __[1]__
* `CA_SUBJECT` CA Subject, default `test-ca`
* `CA_EXPIRE` CA Expiry, default `60` days
* `SSL_CONFIG` SSL Config, default `openssl.cnf` __[1]__
* `SSL_KEY` SSL Key file, default `key.pem`
* `SSL_CSR` SSL Cert Request file, default `key.csr`
* `SSL_CERT` SSL Cert file, default `cert.pem`
* `SSL_SIZE` SSL Cert size, default `2048` bits
* `SSL_EXPIRE` SSL Cert expiry, default `60` days
* `SSL_SUBJECT` SSL Subject default `example.com`
* `SSL_DNS` comma seperate list of alternative hostnames, no default [2]
* `SSL_IP` comma seperate list of alternative IPs, no default [2]

__[1] If file already exists will re-use.__
__[2] If `SSL_DNS` or `SSL_IP` is set will add `SSL_SUBJECT` to alternative hostname list__

Examples
--------

### Create Certificates for NGINX

Enable SSL in `/etc/nginx/sites-enabled/default`:

```shell
server {
        listen 443;
        server_name blog.andradaprieto.es;
        root html;
        index index.html index.htm;
        ssl on;
        ssl_certificate /etc/nginx/certs/blog.andradaprieto.es-cert.pem;
        ssl_certificate_key /etc/nginx/certs/blog.andradaprieto.es-key.pem;
        ssl_session_timeout 5m;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

Restart NGINX and test:
```shell
$ service nginx restart
$ echo '127.0.2.1       blog.andradaprieto.es' >> /etc/hosts
$ curl --cacert /etc/nginx/certs/ca.pem https://blog.andradaprieto.es
<!DOCTYPE html>
<html>
<head>
...
```

### Create keys for docker registry
```shell
$ docker run -d \
    --name registry \
    --volumes-from certs \
    -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/cert.pem \
    -e REGISTRY_HTTP_TLS_KEY=/certs/key.pem \
    -p 5000:5000 \
    registry:2
```

Make sure it works:
```shell
$ echo "127.0.2.1       test.pruebas.local" >> /etc/hosts
$ docker tag jorgeandrada/ca-certificates test.pruebas.local:5000/jandrada
$ docker push test.pruebas.local:5000/jandrada
The push refers to a repository [test.pruebas.local:5000/jandrada] (len: 1)
xxxxxxxxx: Pushed
```
