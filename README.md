# ca-certificates [![](https://images.microbadger.com/badges/version/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own version badge on microbadger.com")  [![](https://images.microbadger.com/badges/image/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/commit/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own commit badge on microbadger.com")
Crea CA y certificados basados en el mismo.

## Ejecución:

### Crear CA, primera ejecución:
```shell
docker run --rm \
  -e SSL_SUBJECT=andradaprieto.es \
  -e SSL_DNS=andradaprieto \
  -v /var/certs:/certs \
  dockreg01.virt.cga/capacidad/ca-virt:latest
```

### Creación de certificados, posteriores ejecuciones:
```shell
docker run --rm \
  -e SSL_SUBJECT=blog.andradaprieto.es \
  -e SSL_DNS=blog \
  -v /var/certs:/certs \
  dockreg01.virt.cga/capacidad/ca-virt:latest
```
