**The first step is to add NGINX official repository :**  
Source: https://nginx.org/en/linux_packages.html
```sh
sudo apt install -y curl gnupg2 ca-certificates lsb-release debian-archive-keyring && \
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
| sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null && \
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/debian `lsb_release -cs` nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list
```
**You should now be able to install NGINX 1.26.1 :**
```sh
sudo apt update && \
sudo apt install -y nginx=1.26.1-2~$(lsb_release -cs)
```
**Enable and and start nginx**
```sh
sudo systemctl enable nginx
sudo systemctl start nginx
```
**Verify the version**
```sh
nginx -v
```
