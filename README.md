podman run -it --rm \
    -v /home/vagrant/.local/share/containers/storage/openbao-data:/var/lib/openbao:Z \
    -v /home/vagrant/certs:/certs:Z \
    -e BAO_SEAL_TYPE=kmip \
    -e BAO_KMIP_WRAPPER_KEY_ID=1 \
    -e BAO_KMIP_ENDPOINT=127.0.0.1:5696 \
    -e BAO_KMIP_CA_CERT=/certs/ca.crt \
    -e BAO_KMIP_CLIENT_CERT=/certs/client.crt \
    -e BAO_KMIP_CLIENT_KEY=/certs/client.key \
  docker.io/openbao/openbao:latest \
  sh
  
  podman volume create openbao-certs

  podman run \
    --name openbao1 \
    --detach \
    -v /home/vagrant/.local/share/containers/storage/openbao-data:/var/lib/openbao \
    -v /home/vagrant/certs:/openbao/certs \
    -e BAO_SEAL_TYPE=kmip \
    -e BAO_KMIP_WRAPPER_KEY_ID=1 \
    -e BAO_KMIP_ENDPOINT=127.0.0.1:5696 \
    -e BAO_KMIP_CA_CERT=/openbao/certs/ca.crt \
    -e BAO_KMIP_CLIENT_CERT=/openbao/certs/client.crt \
    -e BAO_KMIP_CLIENT_KEY=/openbao/certs/client.key \
    -e BAO_KMIP_ENCRYPT_ALG=RSA_OAEP_SHA512 \
    -e BAO_KMIP_TLS12_CIPHERS=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 \
    --publish 8200:8200 \
    docker.io/openbao/openbao:latest



podman run \
  --name openbao1 \
  --detach \
  -v /usr/lib64/pkcs11/libsofthsm2.so:/usr/lib64/pkcs11/libsofthsm2.so \
  -e BAO_SEAL_TYPE=pkcs11 \
  -e BAO_HSM_LIB=/usr/lib64/pkcs11/libsofthsm2.so \
  -e BAO_HSM_SLOT=886786069 \
  -e BAO_HSM_TOKEN_LABEL=OpenBao \
  -e BAO_HSM_PIN=1234 \
  -e BAO_HSM_KEY_LABEL=bao-root-key-rsa \
  --publish 8201:8200 \
  --replace \
  docker.io/hashicorp/vault

podman exec openbao1 vault version


