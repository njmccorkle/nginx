# nginx-certbot-dns-cloudflare

nginx-certbot-dns-cloudflare is a docker-compose for creating an nginx server with certbot/dns-cloudflare for LetsEncrypt SSL Certificates. It uses dns-cloudflare so that we can get a wildcard cert using the v2 DNS verification method. I mount an NFS share on my Docker host to host volumes for my containers, so all paths are relative to that. Adjust as needed for your scenario.

In my case I'm using this to expose access some of my locally-hosted services out in the wild, so the nginx config is specific to my needs.

## Installation

* Clone the repo into the location of your choice.
* Create a ```cloudflare.ini``` that contains your Cloudflare email address and Global API Key. This will be used by the certbot container do DNS verification when creating and renewing certificates.
```bash
        # Cloudflare API credentials used by Certbot for requesting and renewing SSL keys
        dns_cloudflare_email = email@domain.com
        dns_cloudflare_api_key = 23467gflj4h56o87dvhtr
```
* Run certbot to generate the certificates
```bash
        sudo docker run -it --rm --name certbot \
                -v  /mnt/docker/nginx/letsencrypt:/etc/letsencrypt \
                -v  /mnt/docker/nginx/letsencrypt/lib:/var/lib/letsencrypt \
                -v  /mnt/docker/nginx/letsencrypt/cloudflare.ini:/mnt/docker/nginx/letsencrypt/cloudflare.ini \
                certbot/dns-cloudflare certonly \
                --dns-cloudflare-credentials  /mnt/docker/nginx/letsencrypt/cloudflare.ini \
                --email email@example.com \
                --agree-tos \
                --server https://acme-v02.api.letsencrypt.org/directory \
                -d domain.com \
                -d *.domain.com

```
* If needed, the certificate can be renewed manually by running the following. This should not be needed since it should be handled automatically by the container created in compose.
```bash
        sudo docker run -it --rm --name certbot \
                -v  /mnt/docker/nginx/letsencrypt:/etc/letsencrypt \
                -v  /mnt/docker/nginx/letsencrypt/lib:/var/lib/letsencrypt \
                -v  /mnt/docker/nginx/letsencrypt/cloudflare.ini:/mnt/docker/nginx/letsencrypt/cloudflare.ini \
                certbot/dns-cloudflare renew

```

## Usage
After the certificate has been generated we can run the compose file.
```bash
docker-compose up -d
```
