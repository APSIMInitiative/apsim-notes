# apsim.info SSL certificates

Currently ```*.apsim.info``` certificates are signed by GoDaddy. Steps to renew them:

1. Download the certificates from GoDaddy to apsim.info FTP site (e.g. ftp://apsimdev.apsim.info/ApsimX/Documents/apsim.info.zip) - temporarily.
2. Log into dev.apsim.info: ```ssh webmaster@dev.apsim.info```
3. Download the certificates zip file from the ftp site to ~/sources/apsim-web/httpd: 
```
cd ~/sources/apsim-web/httpd.certs
sudo curl --output apsim.info.zip https://apsimdev.apsim.info/ApsimX/Documents/apsim.info.zip
```
4. Delete the temporary zip file from the FTP site.
5. Unzip the zip file. ```unzip apsim.info.zip```
6. Rename gd_bundle-g2-g1.crt to server-chain.crt 
7. Rename a9978d9d50e38ed9.crt to server.crt
9. To rebuild docker container and restart httpd (apache2) container, run
```
cd ..
./deploy
``` 
