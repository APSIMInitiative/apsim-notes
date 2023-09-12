# apsim.info SSL certificates

Currently ```*.apsim.info``` certificates are signed by GoDaddy. Steps to renew them:

1. Download the certificates from GoDaddy
2. SCP them to dev.apsim.info: scp * webmaster@dev.apsim.info:~/sources/apsim-web/httpd/.certs
3. Log into dev.apsim.info: ```ssh webmaster@dev.apsim.info```
4. cd sources/apsim-web/httpd/.certs
5. Rename gd_bundle-g2-g1.crt to server-chain.crt 
6. Rename a9978d9d50e38ed9.crt to server.crt
7. To rebuild docker container and restart httpd (apache2) container, run
```
cd ..
./deploy
``` 
