# This is an example of Snapshot. Substituti it with your owen information

# Make sure you are running the node you want to make api+RPC for

# 1. Have a Domain
- **Buy a Domain at [idcloudhost.com](https://idcloudhost.com/) or wherever it is**

# 2. Connecting the domain to cloudflare

**a. Add site**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/foto/2.png)

- **Enter https://cloudflare.com/**
- **Then register your account to cloudflare**
- **then hit add a site**

**b. Register domain**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/foto/3.jpg)

- **Enter the domain that you bought in the format above without http**
- **Then press add site**

**c. Select Plan**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/foto/4.png)

- **In the payment section, select the free one** 
- **Then press continue**

**d. Review Dns Record**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/foto/5.png)

- **In this section, just continue**

**e. Change your nameservers**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/foto/6.png)

- **First open the provider where you bought the domain then**
- **press manage on your domain**
- **then hit nameservers**
- **Copy the server name that Cloudflare has provided, then move it to your provider**
- **like the picture above**
- **and wait a few moments to connect**

**f. Setting DNS**
![Image alt](https://github.com/Node-max/HOW-TO-MAKE-RPC-API/blob/main/foto/7.jpg)

- **press dns**
- **then press record**
**g. Create a Subdomain**
**1. Create dns TYPE A**
<img width="506" alt="image" src="https://user-images.githubusercontent.com/61777095/224560581-1b0bff40-8c52-4449-8554-101eb104f97e.png">

- **1.Press Add Records**
- **2.Choose Type A**
- **3.Fill in the Name nolustest**
- **4.IPV4 ADDRES Is your vps ip address**
- **5.turn off status proxies**
- **6.Then Save**

**2. Creating an SNAPSHOT Subdomain**
<img width="937" alt="image" src="https://user-images.githubusercontent.com/61777095/224560537-852bee44-534b-4d34-ba49-db4895547245.png">
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

**b. Create a Folder For Snapshots**
I recommend using mobaxtream for this section to make it easier
**1. Press Back Button**
<img width="444" alt="image" src="https://user-images.githubusercontent.com/61777095/224560821-a342f62c-36d0-473d-a936-06e6a208e2f9.png">

**a. Login to your vps**
**b. Press the folder section back**
**2. Go to the Var folder**
<img width="474" alt="image" src="https://user-images.githubusercontent.com/61777095/224560968-71d1888a-b437-4b7e-ae65-45ce3e9732fc.png">

**3. Create a www folder**
<img width="475" alt="image" src="https://user-images.githubusercontent.com/61777095/224561021-b25d03ba-6cd2-4163-9071-a0b99edce1de.png">

**a. Press Pictures Folder +**
**b. Enter Folder Name www**

**4. Create Snapshot Folders**
<img width="436" alt="image" src="https://user-images.githubusercontent.com/61777095/224561386-f08624d3-a37b-4f98-a3bd-8b08b03c5ffa.png">

**a. Press Pictures Folder +**
**b. Enter the Snapshot Folder Name**

**5. Create a Nolus Folder**
<img width="450" alt="image" src="https://user-images.githubusercontent.com/61777095/224561471-ffa191cd-db81-48b9-9b03-585e2d7daaa7.png">

**a. Press Pictures Folder +**
**b. Enter the Snapshot Folder Name**

**4. Create Snapshot Files**
```pytho
sudo apt install lz4
cd $HOME/.nolusd
sudo systemctl stop nolusd
```
```python
tar -cf - data | lz4 > /var/www/snapshot/nolus/nolus-snapshot-$(date +%Y%m%d).tar.lz4
```
<img width="470" alt="image" src="https://user-images.githubusercontent.com/61777095/224561710-bcd0739d-63d4-400a-ace3-1729b4958cbc.png">

**a. The process is long because the snapshot file is already zipped**
**b. If you can type the command it means it's done**

**5. Create a Config Snapshot**
Please Follow The Following Instructions
```python
nano /etc/nginx/sites-enabled/snapshot.planq.spt-node.xyz.conf
```
**Enter the following script and change it a little like the image below**
```python
server {
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name <your_domain_name>; 


        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                root /var/www/snapshot/;
                autoindex on;
        } 
}
```
<img width="488" alt="image" src="https://user-images.githubusercontent.com/61777095/224607532-221f3ca6-d340-400d-a7e1-17bc7a75ac05.png">

**a. Don't forget to remove the <> sign**
**b. Press ctrl+o to save, ctrl+c to exit**

#6. 
a. Installing ssl
```python
sudo certbot --nginx --register-unsafely-without-email
sudo certbot --nginx --redirect
```
**Select the required subdomain that needs to be registered**
#7. Run nginx
```python
sudo systemctl start nginx
```
#8. Check snapshots
Enter your browser then check the subdomain that you created earlier like


