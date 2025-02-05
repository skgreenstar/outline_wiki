# Docker compose for outline wiki


Features:

* A simple make and interactive bash script to help you generate all the conf required
* A docker-compose to run your service
* Dummy https certificate generator. Replace with your certificates after generation
* Use minio instead of AWS S3, so that everything is really self-hosted
* nginx reverse proxy for outline and minio

Runs the outline server with https if required

# How to use 

```
cd outline-wiki-docker-compose
make install
```

And follow the instructions.


서버에서 ssl 인증서 발급 및 셋팅 방법

```
docker run --rm \
  --network docker-compse와 동일한 네트워크 \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/www/certbot:/var/www/certbot \
  certbot/certbot certonly \
  --webroot -w /var/www/certbot \
  --email your-email@example.com --agree-tos --no-eff-email \
  --force-renewal -d 도메인
```


인증서 생성 확인

```
ls -al /etc/letsencrypt/live/도메인/

```

🔹 Nginx가 인증서를 읽을 수 있는지 확인
✅ fullchain.pem, privkey.pem 파일이 보여야 합니다. 
```
docker exec -it nginx ls -al /etc/letsencrypt/live/도메인/
```


✅ 5. SSL 인증서 자동 갱신 설정
Let's Encrypt SSL 인증서는 90일마다 갱신해야 합니다.
자동 갱신을 설정하려면 **크론 작업(cron job)**을 추가해야 합니다.

🔹 갱신 테스트
```
docker run --rm \
  --network docker-compse와 동일한 네트워크\
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/www/certbot:/var/www/certbot \
  certbot/certbot renew --dry-run
```

🔹 자동 갱신 추가 (크론탭 설정)
```
crontab -e
```

그리고 아래 줄을 추가하여 매일 새벽 3시에 자동 갱신하도록 설정합니다.

```
0 3 * * * docker run --rm --network docker-compse와동일한 네트워크 -v /etc/letsencrypt:/etc/letsencrypt -v /var/www/certbot:/var/www/certbot certbot/certbot renew --quiet && docker restart nginx
```


