# ca-certificates
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
