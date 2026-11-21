Ansible Role for Openbao
=========

Example Requirements
--------------------

```yml
---
collections:
  - community.crypto
  - containers.podman

roles:
  - name: cert-generator
    src: git+https://github.com/moukail/ansible-role-cert-generator.git
    version: main
  - name: openbao.podman
    src: git+https://github.com/moukail/ansible-role-openbao.podman.git
    version: main

```

Example Playbook
----------------

```yaml
---
- name: Install Openbao
  hosts: all
  become: true

  vars:
    cert_ca_passphrase: "" # REQUIRED: provide through Ansible Vault (minimum 16 chars)
    cert_ca_dir: /etc/pki/tls/
    cert_ca_certificate_path: /etc/pki/tls/ca.crt

    openbao_port: 8200

  tasks:
    - name: Generate certificate
      ansible.builtin.include_role:
        name: cert-generator
      vars:
        cert_dir: /opt/openbao/tls
        cert_certificate_path: /opt/openbao/tls/server.crt
        cert_key_path: /opt/openbao/tls/server.key

    - name: Run Openbao with Podman
      ansible.builtin.include_role:
        name: openbao.podman

  post_tasks:
    - name: Show Openbao Keys
      ansible.builtin.debug:
        msg:
          - "Openbao address: https://{{ ansible_facts['default_ipv4']['address'] }}:{{ openbao_port }}"
          - "Openbao token: {{ openbao_token }}"
          - "Openbao keys: {{ openbao_keys }}"
      when: openbao_token is defined and openbao_keys is defined

```


```bash

podman run \
  --name openbao \
  --replace \
  -p 8200:8200 \
  -v "$(pwd)/config:/openbao/config:ro,Z" \
  -v "$(pwd)/tls:/openbao/tls:ro,Z" \
  -v "$(pwd)/data:/openbao/data:U,Z" \
  docker.io/openbao/openbao:2.7.2 \
  server

podman run \
  --name openbao \
  --replace \
  -p 8200:8200 \
  -v "$(pwd)/config:/openbao/config:ro,Z" \
  -v "$(pwd)/tls:/openbao/tls:ro,Z" \
  -v "$(pwd)/data:/openbao/data:U,Z" \
  -v /etc/softhsm2.conf:/etc/softhsm2.conf:ro \
  -v /opt/softhsm/tokens:/opt/softhsm/tokens \
  -v /lib/softhsm/libsofthsm2.so:/lib/softhsm/libsofthsm2.so:ro \
  quay.io/openbao/openbao-ubi:2.7.2 \
  server

mkdir -p /opt/pykmip/data
mkdir -p /opt/pykmip/logs

podman run -d \
  --name pykmip \
  --replace \
  -p 5696:5696 \
  -v /opt/pykmip/config/pykmip.conf:/etc/pykmip/pykmip.conf:ro,Z \
  -v /opt/pykmip/certs:/etc/pykmip/certs:ro,Z \
  -v /opt/pykmip/policies:/etc/pykmip/policies:ro,Z \
  -v /opt/pykmip/data:/var/lib/pykmip:Z \
  -v /opt/pykmip/logs:/var/log/pykmip:Z \
  localhost/pykmip-server:latest

podman run -it --rm \
  --name openbao \
  -p 8200:8200 \
  -p 8201:8201 \
  -v "$(pwd)/config/config.hcl:/openbao/config.hcl:ro,Z" \
  -v "$(pwd)/data:/openbao/data:Z" \
  -v "$(pwd)/tls:/openbao/tls:ro,Z" \
  docker.io/openbao/openbao:latest \
  server \
  -config=/openbao/config.hcl
  
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

```
