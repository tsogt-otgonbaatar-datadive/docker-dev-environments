# Docker and local environment

This repo includes `traefik` and `docker-ssh-tunnel`

## Generate custom SSL certificate
We will use `mkcert` for generating SSL certificates.
[https://github.com/FiloSottile/mkcert](https://github.com/FiloSottile/mkcert)

1. `mv conf/tls_certificate.local.yml.example conf/tls_certificate.local.yml`
2. Store generated certificates in `certs` folder.
3. Add newly created certicates in `conf/tls_certificate.local.yml`
4. `docker-compose up -d`

## Configure `dnsmasq`
[https://www.adaltas.com/en/2022/11/17/traefik-docker-dnsmasq/](https://www.adaltas.com/en/2022/11/17/traefik-docker-dnsmasq/)

Mac:
```
# Install dnsmasq
brew install dnsmasq
# Setup the `.local` pseudo-TLD
mkdir -pv $(brew --prefix)/etc/
echo 'address=/.local/127.0.0.1' >> $(brew --prefix)/etc/dnsmasq.conf
# Start dnsmasq
sudo brew services start dnsmasq
# Add to resolver
sudo mkdir -v /etc/resolver
sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/local'
# Test
scutil --dns
```

Linux instructions with NetworkManager (eg Ubuntu Desktop):
```
# Disable systemd-resolved
systemctl disable systemd-resolved
systemctl stop systemd-resolved
unlink /etc/resolv.conf
# Activate the dnsmasq plugin
cat <<CONF | sudo tee /etc/NetworkManager/conf.d/00-use-dnsmasq.conf
[main]
dns=dnsmasq
CONF
# Setup the public DNS and the `.local` pseudo-TLD
cat <<CONF | sudo tee /etc/NetworkManager/dnsmasq.d/00-dns-public.conf
server=8.8.8.8
CONF
cat <<CONF | sudo tee /etc/NetworkManager/dnsmasq.d/00-address-local.conf
address=/.local/127.0.0.1
CONF
systemctl restart NetworkManager
```