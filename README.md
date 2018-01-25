# ca-certificates [![](https://images.microbadger.com/badges/version/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own version badge on microbadger.com")  [![](https://images.microbadger.com/badges/image/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/commit/jorgeandrada/ca-certificates:latest.svg)](https://microbadger.com/images/jorgeandrada/ca-certificates:latest "Get your own commit badge on microbadger.com")
Crea CA y certificados basados en el mismo.

## Ejecución:

### Crear CA, primera ejecución:
```shell
docker run --rm \
  -e SSL_SUBJECT=andradaprieto.es \
  -e SSL_DNS=andradaprieto \
  -v $(pwd)/certs:/certs \
  jorgeandrada/ca-certificates:latest
```

### Creación de certificados, posteriores ejecuciones:
```shell
docker run --rm \
  -e SSL_SUBJECT=blog.andradaprieto.es \
  -e SSL_DNS=blog \
  -v $(pwd)/certs:/certs \
  jorgeandrada/ca-certificates:latest
```

=================


Advanced Usage
--------------

Customize the certs using the following Environment Variables:

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

_Creating web certs for testing SSL just got a hell of a lot easier..._

Create Certificate:
```
$ docker run -v /tmp/certs:/certs \
  -e SSL_SUBJECT=test.example.com   paulczar/omgwtfssl
```

Enable SSL in `/etc/nginx/sites-enabled/default`:

```
server {
        listen 443;
        server_name test.example.com;
        root html;
        index index.html index.htm;
        ssl on;
        ssl_certificate /tmp/certs/cert.pem;
        ssl_certificate_key /tmp/certs/key.pem;
        ssl_session_timeout 5m;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

Restart NGINX and test:

```
$ service nginx restart
$ echo '127.0.2.1       test.example.com' >> /etc/hosts
$ curl --cacert /tmp/certs/ca.pem https://test.example.com
<!DOCTYPE html>
<html>
<head>
...
```


### Create keys for docker registry

_Slightly more interesting example of using `paulczar/omgwtfssl` as a volume container to build and host SSL certs for the Docker Registry image_

Create the volume container for the registry from `paulczar/omgwtfssl`:

```
$ docker run \
  --name certs \
  -e SSL_SUBJECT=test.example.com \
  paulczar/omgwtfssl
----------------------------
| OMGWTFSSL Cert Generator |
----------------------------

--> Certificate Authority
====> Generating new CA key ca-key.pem
Generating RSA private key, 2048 bit long modulus
..........+++
.......................................................+++
e is 65537 (0x10001)
====> Generating new CA Certificate ca.pem
====> Generating new config file openssl.cnf
====> Generating new SSL KEY key.pem
Generating RSA private key, 2048 bit long modulus
........................................................................................................................................................+++
...+++
e is 65537 (0x10001)
====> Generating new SSL CSR key.csr
====> Generating new SSL CERT cert.pem
Signature ok
subject=/CN=test.example.com
Getting CA Private Key
```

Run the registry using `--volumes-from` to use the volume container created above:

```
$ docker run -d \
    --name registry \
    --volumes-from certs \
    -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/cert.pem \
    -e REGISTRY_HTTP_TLS_KEY=/certs/key.pem \
    -p 5000:5000 \
    registry:2
```

Make sure it works:
```
$ echo "127.0.2.1       test.example.com" >> /etc/hosts
$ docker tag paulczar/omgwtfssl test.example.com:5000/omgwtfbbq
$ docker push test.example.com:5000/omgwtfbbq
The push refers to a repository [test.example.com:5000/omgwtfbbq] (len: 1)
e34964fe7cfa: Pushed
d52b82eb9ff3: Pushed
6b030e7d76a6: Pushed
8a648f689ddb: Pushed
latest: digest: sha256:8a97202b0ad9b375ff478d84ed948ae7ddd298196fd3b341fc8391a0fe71345a size: 7617
```

### Generate Keys for Kubernetes Secret:

```
$docker run -ti --rm -e OUTPUT=k8s -e SSL_SUBJECT=test.example.com paulczar/omgwtfssl
----------------------------
| OMGWTFSSL Cert Generator |
----------------------------

--> Certificate Authority
====> Generating new CA key ca-key.pem
Generating RSA private key, 2048 bit long modulus
..+++
........................+++
e is 65537 (0x10001)
====> Generating new CA Certificate ca.pem
====> Generating new config file openssl.cnf
====> Generating new SSL KEY key.pem
Generating RSA private key, 2048 bit long modulus
...................................+++
................................................................................................................+++
e is 65537 (0x10001)
====> Generating new SSL CSR key.csr
====> Generating new SSL CERT cert.pem
Signature ok
subject=/CN=example.com
Getting CA Private Key
====> Complete
keys can be found in volume mapped to /certs

====> Output results as base64 k8s secrets
---
apiVersion: v1
kind: Secret
metadata:
  name: omgwtfssl
  namespace: default
type: kubernetes.io/tls
data:
  ca_key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBcysrd2p1d00vSG41MmxCQm0rSU84OW9INkwvaTd0QnBlUHdrd29uNUlqcGVQVHZzCkdYRGx1eXFlZUd3bnlIanZUYTlaQ2luNHd6dkk0Mzg5NjdMSVFCQ0krWlJwRmxtYTl3T3dLOFU1R0lnNWExUVgKa21lbEpyUkQ4QVlKeW1kNmpJL0J5ejZKVXI5WlNkMWxRUXdteWM0bEJZMC9YZVJnWVZobk5GOXZ3dmdJeTdiVgptMjRjSTE2QzdDbHpxcVNwOXhucElOdlBwejhSWVNGRkxXdmdvRUI0REhjeWxWK1lROFlSSUh1TEQ1bVM3WDZJCnlYT0d2RHVtWGVSbWRQWE42VTRKMkhzUk9aQkE4dHRNMGw1YmVoMWtKUjdGQ3I4N1lKaFJQbXpid2lCM1g2dUsKOE1hV0dRcWdtTnFOeVBIc0xSZjJGcXg5UFJxVmRRNmNxY1BscndJREFRQUJBb0lCQUFVY0NGSjJwNE8vM3ZWeApFL3ZlTm5oOE4zOUhlNlUyYTloUXFLYVJqbTZIWThldlhPdjRCYjREM3M0aW5CcVlQSXRqTUU4V2xBYlBPc3dpCi85b2lrSWNwTVFxTlNWS21KcjFlOEhDOXYvcFBXL29OUUVJYkNWaUpBK2piOHdrNVdRU0d6SVQ5K0o1TjZyWXIKUUVYUmw0UmhnekdlS2k5N1hiVkY4bUJOV1hvYXpzU3E5c3VMRUZLMkNxaGtDMWJvb0RJQ1JzYXVDOTlyMFVzVgpZVUYvOWc3VUJzdG9VNmFMTkQvOGdOSDlDblFxNVdMWUNHbXg1cWpudUtYZnFkK0tseXVwakNIWlNhME5uNFRrClFINFQ5QjdIVTRRMkE1M29uV0xXQWlhM3o4Q1lHK1c5M2RzK3A4czEzSzNrSG1BNGU4QTB1bWJvMDkwWk9VRnEKZDRqa2U2RUNnWUVBNXBSMzJOZHJNbmlOdlhGcHJvQVNLZ2VmOGhvV2hzRTQyQmROWTlJajdVMmRpYjUzNWpCcwpBUUVhalRCdU41V3F6bE5JWS9aWURlQUtLUzR2elFqTzN4aUdXeHNUbDlmYVZLMVRrQU1iNFlCdEJPcjhxU2tGClIyWTkwMWdNSDdxSTQ1NXR2UUh4dms5ZDdaVmYwK3MvS0dsaCtreENkeUNTY0loOFErUVZ5eDhDZ1lFQXg4WHUKM1E2V3E4dkhwRVNYWFZDTE1kbnp2RXRNMStXbmUrR2FEdWZXaGlEM0szV1pGVUFhOWlha1FGUHdweFdhWGdTZwpLdW9zbmsyZzFqNWhyMmRwUnRISUg1OWJ0WllVZkYwNDNORFpXbCtQVVI4OGFCblJ0ZUJNNEFTbGN6UlJmY1FHCmJEL3I4cUZxcjNpU05rL0JXWnhOZmRYTTBzL0MxWGE0ZFRLZ0kzRUNnWUIyU1N2Q0hhQnNYOU8ybjN1cmZSL1UKVjkwQmdjaVVrSUxzdCtlSGtjSEVkdENlWFF0OUZ0SVJJZFBSSWtzb1VLc3k5UjVweVhJYWpCZ3FUL3RObjNzZwpqNkE5RklMVW5uTHVoWXdja0x1NHp2MGVUTDRZdVdadjNrOVJJQlg0SU9VZ2Y2R2tHRjgvVmMvRmxaOTNRM00rCjgvRERTbU8rWVFNK240Vy8vajMvMlFLQmdRQ0s0bWFQdEdhM2hQS0VsMU1NQXNUaW9YMjd1RFh5R3F6M2lQNUwKd28zM3JjOW9uVmNSMlFGbGc3UEpMUkl2ZHV0YzFhWnNiMlVab1NwaUIvaHRzMTBUYVNEU0t6M2MzanZid2d1WQpLSElUVEVBY3k3UjVRd255Z2Ird05rcTM3dldBazlsTFJKMUtqMEhXUCtLV2M1Z2VMbllKTjZ3d0cxUitod3p5CkhZSUxZUUtCZ1FEYTY3b0VPUFRySU5Kdjd4Nk92ck11Vmk2N0JnajhYa0lNZWtNVUtrZVJDZDBSQkV0Z3ArQS8KM0k4OVE0c3NHZWRIay9nUWt4bXlTc0FrMXFLQnl1UDBmK3dIenpzNDlsaENOYUxGeUtuZnlvMkVlSVpvK21COApnRjFVZGNHZkZSalFtWXNCaVJKSEpDS01WZ2RMWFdha3pJMGU5V21BL2g1RDdSWlZNQlFwUEE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
  ca_cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5ekNDQWQrZ0F3SUJBZ0lKQUtyN0N3czdEQWthTUEwR0NTcUdTSWIzRFFFQkN3VUFNQkl4RURBT0JnTlYKQkFNTUIzUmxjM1F0WTJFd0hoY05NVGN4TVRFd01UVXpOak15V2hjTk1UZ3dNVEE1TVRVek5qTXlXakFTTVJBdwpEZ1lEVlFRRERBZDBaWE4wTFdOaE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBCnMrK3dqdXdNL0huNTJsQkJtK0lPODlvSDZML2k3dEJwZVB3a3dvbjVJanBlUFR2c0dYRGx1eXFlZUd3bnlIanYKVGE5WkNpbjR3enZJNDM4OTY3TElRQkNJK1pScEZsbWE5d093SzhVNUdJZzVhMVFYa21lbEpyUkQ4QVlKeW1kNgpqSS9CeXo2SlVyOVpTZDFsUVF3bXljNGxCWTAvWGVSZ1lWaG5ORjl2d3ZnSXk3YlZtMjRjSTE2QzdDbHpxcVNwCjl4bnBJTnZQcHo4UllTRkZMV3Znb0VCNERIY3lsVitZUThZUklIdUxENW1TN1g2SXlYT0d2RHVtWGVSbWRQWE4KNlU0SjJIc1JPWkJBOHR0TTBsNWJlaDFrSlI3RkNyODdZSmhSUG16YndpQjNYNnVLOE1hV0dRcWdtTnFOeVBIcwpMUmYyRnF4OVBScVZkUTZjcWNQbHJ3SURBUUFCbzFBd1RqQWRCZ05WSFE0RUZnUVVYa0JJdm9CdTA5OW1DdEovClkyR29NamJ2T3BFd0h3WURWUjBqQkJnd0ZvQVVYa0JJdm9CdTA5OW1DdEovWTJHb01qYnZPcEV3REFZRFZSMFQKQkFVd0F3RUIvekFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBSCt0WGVlYXpLZ2lqd0NxRlNHbk9LcktxalYwRQpGVVJscWsrdUMzQ1lsdHRWSWV0eGlZbG50Vm00MlNnQkhWTXJZMmJxa1plM1RiaVZSbnp5UzFUa2I0dkpTY0lKCjhKUGtEQ2c4OVhxMlpMdm85dzJMZ3diWWJCczFjZUxiT3c1Y3haOVdwaDQrUnhrTmpocU1jTEJna3RZbVJUQ2MKQUx3T282WjB1NGJsckM4eE00ak53TmptQU96WTYrbTFPU0FPU28zWW9EQXdGenR4NHpWc1V4MmRPR09IMVFsSwpFWVVSRitMQXN6WnJvakMzbko0YVduc04vNEpFWU1CUmlJUVBMUWVDTjJGU2kxaXp2VzZEdW9PdGpIQkJYcnJCCnlOSGk0ZXVkUk1hRFJrS3RydXRqcUFhYWdOdndyZ0tmREdBdW9WNFhKRS93R0tIVE5HaGJzaEZrbEE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  ssl_key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBeWxxejExMW5oMkdSR1gwSTFoVnIrd1hOKzRCM2o4b2QwK2xYWmdwUVV0Wms3LzlHCktSckMvTXZMZCsvcThHM1NIT1NKNjJwa1E4TlpNMml5Z2Fmd1NrRTZNRGZLZUhQdzZoMG95STEwRUQ0WDcvL1IKZFRRZ2pMS29laTQ5bnNGQ0pLSWhsVTJUZzlPeW1hVTY4Zm01OVpsWjJNdU5PWXg1TWtYODRPeXh5Y3VzUHYyVwp1WTN2emg5ZW9LakNKWVRqNjVaZDFPUlp6VmdlOUg3K3ZFa3E1L2tVcm9DSExudS9HSWthdHh3dzFZZ09oV2UrCnZjMmIxc2ZraHZ6ZzVHdFdJNjdlU1dMRE93ZTU3dStFYnc4Tk1QUkZTTmFwWnMyaDkzUGRPb1k0SjdrdVRKaEYKYnpCQVpxWUxyNHhoOWFkazlhT3djUXhaOWZyZ2w3V3NtQUo0aFFJREFRQUJBb0lCQUVtNGFiU1lidE0vbEdFNAowRE5PY3B4dExQSG5oNmQyOXc3dy84cEpENkplUVQrK1BJMVZGcHlLa29JTGdnZzl1a0tVY0RxMzgvZEwvLzllCnNVblRLUk9rVjlLaFFMNUpYd2VyditQd3hNdjJFenA0ZEpMQ2Z1bERYV1hGMnVVdmV5MWxjOTYwK0gwYWJ3TFMKMDNxOGhDY1p5MFhVUW9zb3hpNnVtTVpJOVN5U3hSS1pFdkYvZmJZeVZ6dVpmZGFtYUhPRCttelBZdFdGSWdrbAo4Q0pScGNZeEtPNkdaQUdoa3lEMGtCRlZzSitDdStheWdvTzNXU2dQWXp1eWdrS2tCL05NanFiWE5UaWJlbjRmCmNoeGxmZjRXQTRuRXZVT0VaNm90SHpONFFIUUdKWlliTEJrZkY5K256TGdkZk45S3M5aVNRR0RTOWZrOWl2SSsKbkt3azl6RUNnWUVBN3Jxay9tTTVWMnYyTDZXd0tKLzl5RkJzSGg0dy9IZ1FHWTFucWkzaW9odEtNZHlyVUl1ZApNa1hVa3FWOEZIMGtlOFdBakt3Skt6QVVudkJ6Wm54dWZINTMwN3V0K3B4RjliU1RnK2tNeS81d3F3VWdPUTNqCmlIdytSMGF6MjJKcUZEQnNxUGZMM0c3TllKN3dzVFpZV2dCeGZZK1pCT2ZFME1GVnNsSlBxVHNDZ1lFQTJQNWkKRG8xMWFjbm9NVWFJVFlXblhwT3lwOWFCR25YcmJoeUNQS29zU29jTFJ0YWt3L1BwUTVYdmsrSzZ6eUl2WW5FdApaQUttbEprT1ZNZkJDbE90VHJ6TVVIZkZOWTdzY3pIanZhbVJRdXJaQk1JazdldmY1cUdMQVIrenlLeTB4ZFQzCnZGUkdmSmx6NjNEcFdiZjZ1OVkxVWNVbXZ5aXRyTnFFczVzU1NUOENnWUJveC9rMVFwM2ZkaDUzS1ZVWmI2ZTMKTFQxWE5zOHZjUTgzOSszQkx5U2pIREZEazJTS0ZNMXBUR2NSK3Bwc2I4VDhvbUphM0FPbU5oTkc4NmpqR2NodwowaDJNREhzL1hTb0R2ejlrRFgwMWFEZFJpUTFzbldENS9mWmoyRytHNGpwSEpEMzlKODROc1lCcFlUbXB2bjJtCit6elU3SnN3SVA0czFqN2o1dWJhRXdLQmdIRWpJUWtwWkVDR0QxUXh6RHR2SmphL2wzUysrSTFOVWpVVkZDcUIKSjVxc0Vvc2F1c21ZVU5UMlJmVzdUMTlVR1pTZ0llUjFKVmx3Ky9Ia1BKZ2Z6TXF5MFd4YkppMm9tVXZ1aFNtTQpVYnFzSy82NUl2d1I4YW1VTEorbllkdU5nS3R0UU1XbXd5R1ArTXFYRW5PKzR6SXdtNWhJek16NmJxTWpRL0ZKCk54Mk5Bb0dBRFlpMTI4TENINjJWKzJkd2k5bVE1SUxqdXRRVUlSeWFiNXZ2R1NzemlhUkFWVmQ0MkxyUEF3YWwKWEVJbm54RkJQVnVPc003SXF4blBvdHM4NURBNkMyeGZLRDRMODBYZ0thb0tYamY0a1JNZEdTZEl2ZnRPQ0ZERQpDRklOM2JJY3JhcEZ4Q0EraGNtQzhQTzlFTm5HM1VXWnBSUEhvcVNKS0k5c3g3bG1IVDA9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
  ssl_csr: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ296Q0NBWXNDQVFBd0ZqRVVNQklHQTFVRUF3d0xaWGhoYlhCc1pTNWpiMjB3Z2dFaU1BMEdDU3FHU0liMwpEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUURLV3JQWFhXZUhZWkVaZlFqV0ZXdjdCYzM3Z0hlUHloM1Q2VmRtCkNsQlMxbVR2LzBZcEdzTDh5OHQzNytyd2JkSWM1SW5yYW1SRHcxa3phTEtCcC9CS1FUb3dOOHA0Yy9EcUhTakkKalhRUVBoZnYvOUYxTkNDTXNxaDZMajJld1VJa29pR1ZUWk9EMDdLWnBUcngrYm4xbVZuWXk0MDVqSGt5UmZ6Zwo3TEhKeTZ3Ky9aYTVqZS9PSDE2Z3FNSWxoT1BybGwzVTVGbk5XQjcwZnY2OFNTcm4rUlN1Z0ljdWU3OFlpUnEzCkhERFZpQTZGWjc2OXpadld4K1NHL09Ea2ExWWpydDVKWXNNN0I3bnU3NFJ2RHcwdzlFVkkxcWxtemFIM2M5MDYKaGpnbnVTNU1tRVZ2TUVCbXBndXZqR0gxcDJUMW83QnhERm4xK3VDWHRheVlBbmlGQWdNQkFBR2dTREJHQmdrcQpoa2lHOXcwQkNRNHhPVEEzTUFrR0ExVWRFd1FDTUFBd0N3WURWUjBQQkFRREFnWGdNQjBHQTFVZEpRUVdNQlFHCkNDc0dBUVVGQndNQ0JnZ3JCZ0VGQlFjREFUQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFJVmZqNk9nMW9Ra2MKL1I5VjI2VS9QRnhaWk15bDNRb0xRM0kxbnBTWmJKTExuZHNmcjVoOTcwYzZtZzlVRGFmMWRPdlYzMkU2Y0tJbQpWZ1ZSN0ZzRnUrZHJvQkVUd1Y2cXN0OXhNZSsvV3BHb3B6VmNxOEQ2SVEwbWRJUEthMnNtSnlxU0Q1Y1NVaTlKCitnZGROWHdmbGVST0ZvcElxd3pSNGtHc3pMOVZjOXp5UEExc29zTGM4WnBtTU5WVVNUbWN6ZFdxbGszR0I1SHgKRU82NEl1d0lmRkY0d0hNNVNaMlFUMWpsOS9Gc1RWQndZMm9uQXNaVW4xcEhIR2N2SUFUVFE1MGVQNEJYQW4yMQoyT2t4VDV6Wi9SaENJT1F0M21wZlA1Y3piYXJmOHpyVkl6b2xseWNUWWJLbGpMWlB4YzAxT29VR3V6MTVnUzZWCnlCZjZNMTdkUHc9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  ssl_cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1RENDQWN5Z0F3SUJBZ0lKQU8xdmdpeUVxTEdaTUEwR0NTcUdTSWIzRFFFQkN3VUFNQkl4RURBT0JnTlYKQkFNTUIzUmxjM1F0WTJFd0hoY05NVGN4TVRFd01UVXpOak15V2hjTk1UZ3dNVEE1TVRVek5qTXlXakFXTVJRdwpFZ1lEVlFRRERBdGxlR0Z0Y0d4bExtTnZiVENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DCmdnRUJBTXBhczlkZFo0ZGhrUmw5Q05ZVmEvc0Z6ZnVBZDQvS0hkUHBWMllLVUZMV1pPLy9SaWthd3Z6THkzZnYKNnZCdDBoemtpZXRxWkVQRFdUTm9zb0duOEVwQk9qQTN5bmh6OE9vZEtNaU5kQkErRisvLzBYVTBJSXl5cUhvdQpQWjdCUWlTaUlaVk5rNFBUc3BtbE92SDV1ZldaV2RqTGpUbU1lVEpGL09Ec3NjbkxyRDc5bHJtTjc4NGZYcUNvCndpV0U0K3VXWGRUa1djMVlIdlIrL3J4Skt1ZjVGSzZBaHk1N3Z4aUpHcmNjTU5XSURvVm52cjNObTliSDVJYjgKNE9SclZpT3Uza2xpd3pzSHVlN3ZoRzhQRFREMFJValdxV2JOb2ZkejNUcUdPQ2U1TGt5WVJXOHdRR2FtQzYrTQpZZlduWlBXanNIRU1XZlg2NEplMXJKZ0NlSVVDQXdFQUFhTTVNRGN3Q1FZRFZSMFRCQUl3QURBTEJnTlZIUThFCkJBTUNCZUF3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVIQXdJR0NDc0dBUVVGQndNQk1BMEdDU3FHU0liM0RRRUIKQ3dVQUE0SUJBUUFkOVhmd0MwbXU5REI1Tng4TVJBRlpVWVVUeTM1a0xxMzZoYnF1Zmo3N0hTOHgwOEgySGt3cApsT1pSL1ZOeHpoR2NvVWxXNTdFaEdwSnVabEtmajl2SEJ5eU1SRHl0MUZsWFFXa1UzWHJLdjIyRHpWc013RndICndiWjBPemQxZFRpOFNKSGZ3WUVZb2ZMZVZwaHRiMmMxTytMdllsRVJqYXluYkI1dTNiNTRLL2pPeHErRlhnTGUKUmhheEJNOXJDa3psc0xUbk5UT0Q0SzR3NUFaZDhGRyswRTk1R3FjTjZkUFNRVjhJSEdmSnBPZ3J2Z3NaYTNaTgpPMlBIb3NjTUdUVVRQS0FZVzJZV0hjV1dZLzhveTY2Ti9kUlJ3bWRHdVBVckQ4a1grL2V6cGxRVmFHMUkxZTJQCkgwbHJKTnFMbkROR1pTV2twckJONjlPcXFWU21nRGl3Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
```
