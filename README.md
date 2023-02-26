# HOW-TO-MAKE-RPC-API

# This is an example of RPC-API. Substituti it with your owen information

# Make sure you are running the node you want to make api+RPC for
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image.png)

# 1. Have a Domain
- **Buy a Domain at [idcloudhost.com](https://idcloudhost.com/) or wherever it is**

# 2. Connecting the domain to cloudflare

**a. Add site**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image%20(1).png)

- **Enter https://cloudflare.com/**
- **Then register your account to cloudflare**
- **then hit add a site**

**b. Register domain**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide1.JPG)

- **Enter the domain that you bought in the format above without http**
- **Then press add site**

**c. Select Plan**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image%20(2).png)

- **In the payment section, select the free one** 
- **Then press continue**

**d. Review Dns Record**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image%20(3).png)

- **In this section, just continue**

**e. Change your nameservers**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image%20(4).png)

- **First open the provider where you bought the domain then**
- **press manage on your domain**
- **then hit nameservers**
- **Copy the server name that Cloudflare has provided, then move it to your provider**
- **like the picture above**
- **and wait a few moments to connect**

**f. Setting DNS**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide2.JPG)

- **press dns**
- **then press record**

**g. Create a Subdomain**
**1. Create dns TYPE A**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide3.JPG)

- **1.Press Add Records**
- **2.Choose Type A**
- **3.Fill in the Name nolustest**
- **4.IPV4 ADDRES Is your vps ip address**
- **5.turn off status proxies**
- **6.Then Save**

**2. Creating an API Subdomain**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide4.JPG)

- **1.Press add record again**
- **2.select CNAME type**
- **3.Fill name with api.nolus-test**
- **4.target fill with nolustest.max-node.xyz**
- **5.Turn off Proxy Status**
- **6.Press Save**

**3. Create RPC Subdomains**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide5.JPG)

- **1.Press add record again**
- **2.select CNAME type**
- **3.Fill name with rpc.nolus-test**
- **4.target fill with nolustest.max-node.xyz**
- **5.Turn off Proxy Status**
- **5.Press Save**

**4. Configuration on Vps**
# TRY TO INSTALL 1 1 COMMAND DON'T OVERWRITE IMMEDIATELY

```python
sudo apt update && sudo apt upgrade -y
sudo apt install nginx certbot python3-certbot-nginx -y
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get update && apt install -y nodejs git
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn -y
```
- **b. Activate the Api Server And Find The Port**

```python
sudo nano /root/.nolus/config/app.toml
```
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image%20(5).png)

# THIS INFORMATION BELOW IS IMPORTANT

- **Enter the Command Above into your Vps then scroll down and search as shown above change enable = false to enable = true**
- **save well address = "tcp://0.0.0.0:1317"**
- **then restart your node with the following command:**

```python
sudo systemctl restart nolusd
```
- **c. Looking for RPC Ports**

```python
sudo nano /root/.nolus/config/config.toml
```

![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/image%20(6).png)


- **d. Create Config API**

**CORRECT EXAMPLE**

```python
nano /etc/nginx/sites-enabled/api.nolus-test.max-node.xyz.conf
```

**Enter the following script and change it a little like the image below**

```python
server {
    server_name api.nolus-test.max-node.xyz;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass http://ip-node:1337;

    }
}
```
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide6.JPG)

- **e. Create RPC Configs**

# CORRECT EXAMPLE
```python
nano /etc/nginx/sites-enabled/rpc.nolus-test.max-node.xyz.conf
```
- **Enter the following script and change it a little like the image below**
```python
server {
    server_name rpc.nolus-test.max-node.xyz;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

         proxy_pass http://ip-node-kamu-yang-saya-suruh-simpan-baik-baik:1337;

    }
}
```

![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/Slide7.JPG)

- **f.Installing ssl**

```python
sudo certbot --nginx --register-unsafely-without-email
sudo certbot --nginx --redirect
```
**install ssl to the two subdomains above by entering the command above**

- **g. Check api and rpc**

**Enter your browser then check the subdomain that you created earlier like**
# API
```python
https://api.nolus-test.max-node.xyz/
```
# RPC
```python
https://rpc.nolus-test.max-node.xyz/
```




