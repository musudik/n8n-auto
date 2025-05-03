project-root/
├── docker-compose.yml
├── nginx/
│   ├── conf/
│   │   └── thinkeuros.de.conf
│   ├── certbot/
│   │   ├── conf/           # for /etc/letsencrypt
│   │   └── www/            # for /.well-known/acme-challenge
│   └── html/
├── n8n-data/
├── minio-data/
├── baserow-data/


# ✅ NGINX config file for the root domain thinkeuros.de, supporting both HTTP (for Let's Encrypt validation) and HTTPS with reverse proxy to n8n.

Make sure ports 80 and 443 are open on your VPS firewall.

Make sure thinkeuros.de DNS is correctly pointing to your server IP.

You can test renewal manually with:


## Steps to apply it:
1. Save this file as:
nginx/conf/n8n.thinkeuros.de.conf

2. Make sure DNS A record for n8n.thinkeuros.de points to your VPS IP.

3. Run certbot after DNS is working:
docker-compose run --rm certbot

4. Then restart everything:
docker-compose up -d

## Encrypt auto-renewal cronjob
✅ Step 1: Add a Renewal Service to docker-compose.yml

certbot-renew:
    image: certbot/certbot
    container_name: certbot-renew
    entrypoint: "/bin/sh -c"
    command: >
      "trap exit TERM;
      while :; do
        certbot renew --webroot --webroot-path=/var/www/certbot --quiet --deploy-hook 'nginx -s reload';
        sleep 12h;
      done"
    volumes:
      - ./nginx/certbot/conf:/etc/letsencrypt
      - ./nginx/certbot/www:/var/www/certbot
	  


## This container:

## Runs certbot renew every 12 hours

Uses the webroot challenge (so HTTP access must be open)

Reloads NGINX after renewal with nginx -s reload

✅ Step 2: Rebuild and Launch
Run the following commands:
docker-compose down
docker-compose up -d


Then manually run the initial certificate request if you haven't:
docker-compose run --rm certbot	  


Final Advice
Make sure ports 80 and 443 are open on your VPS firewall.

Make sure thinkeuros.de DNS is correctly pointing to your server IP.

You can test renewal manually with:
docker-compose run --rm certbot renew --dry-run



nca-toolkit:
    image: stephengpope/no-code-architects-toolkit:latest
    container_name: nca-toolkit
    ports:
      - "8080:8080"
    environment:
      - API_KEY=thekey
      - S3_ENDPOINT_URL=http://miniio:9000
      - S3_ACCESS_KEY=your_access_key
      - S3_SECRET_KEY=your_secret_key
      - S3_BUCKET_NAME=nca-toolkit
      - S3_REGION=None
    restart: unless-stopped
	
	
## Your services should now be running on the following ports: Service URL's
### n8n 
http://<host-ip>/:5678
### MiniIO 
http://<host-ip>:9001
### Kokoro TTS 
http://<host-ip>:8880/web
### Baserow 
http://<host-ip>:85
### NCA Toolkit 
http://<host-ip>:8080

	
	