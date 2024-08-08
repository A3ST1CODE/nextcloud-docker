## Docker - Nextcloud
## This repository contains Docker Compose configurations for setting up Nextcloud, OnlyOffice, and Nginx Proxy Manager using Docker containers.

## Organized and Separated Configuration
The configuration is organized into folders, each containing a docker-compose.yml file, except for the Scheduling folder, which contains files for scheduling Nextcloud tasks using cron and Systemd.

- **Nextcloud** folder: Contains `docker-compose.yml` with containers for Nextcloud, Mysql, and Redis.
- **OnlyOffice** folder: Contains `docker-compose.yml` for OnlyOffice.
- **Nginx Proxy Manager** folder: Contains `docker-compose.yml` for Nginx Proxy Manager.
- **Cron** folder: Contains Systemd files for scheduling Nextcloud cron jobs.

---
## Prerequisites:
`git`
`docker`
`docker-compose`

You will need at least two domains: one for **Nextcloud** and one for **OnlyOffice**. Optionally, you can have another domain for Nginx Proxy Manager (where all the domains linked to Docker will be configured, with SSL via **Let's Encrypt**). If you don’t want to spend on a domain, you can get one for free here: Freenom.

### Clone the Repository
Start by cloning the repository with:

```
git clone https://github.com/A3ST1CODE/nextcloud-docker.git
```

### Configure Firewall
If you have an active firewall, open ports 80, 81, and 443. If you're using ufw, you can do this with:

```
sudo ufw allow 80,81,443/tcp
```

---
### Setting Up Nginx Proxy Manager
Navigate to the **Nginx Proxy Manager** directory and run the following command to start Nginx Proxy Manager:

```
docker-compose up -d
```

**Access Nginx Proxy Manager**

- Host: 127.0.0.1
- Port: 81

**Login Credentials**
```
Email: admin@example.com
Password: changeme
```

![Captura de tela de 2021-03-28 19-51-11](https://user-images.githubusercontent.com/981368/112770938-ad307a80-8fff-11eb-9eff-55e09b65b94b.png)

After logging in, enter a valid email address and set a password for your user.

*OPTIONAL* - Configuring a domain for Nginx Proxy Manager:

- Forward Hostname / IP: npm-proxy
- Forward Port: 81

In the **SSL** tab, configure as shown below and save:

You should see a screen similar to this:

![Captura de tela de 2021-03-28 20-14-18](https://user-images.githubusercontent.com/981368/112771453-642df580-9002-11eb-9c3a-2a42e5e3dce4.png)

**Setting Up Nextcloud**
Navigate to the **Nextcloud** directory and edit the **db.env** file. Change the password to your desired value and save. Then edit the docker-compose.yml file and change YOU_PASSWORD_REDIS to your desired password and save. Run the following command to start the **Nextcloud**, **Mysql**, **Nginx**, and **Redis** containers:

`cd nextcloud`

`docker compose -f docker-compose up -d`

### Configuring the Domain for Nextcloud in NPM
Go to **Hosts** -> **Proxy Hosts** -> Add **Proxy Host** to add a new **domain**.

Details	Configuration
- Domain Names:	cloud.example.com
- Scheme:	http
- Forward Hostname: / IP	nextcloud
- Forward Port:	80

Enable the other 3 options as shown below:

![Captura de tela de 2021-03-29 00-25-01](https://user-images.githubusercontent.com/981368/112783225-3c9c5480-9025-11eb-81ea-2aa1c9d52caa.png)

In the SSL tab, configure as shown or enable additional options if desired:

![Captura de tela de 2021-03-29 00-26-21](https://user-images.githubusercontent.com/981368/112783351-86853a80-9025-11eb-9e5b-150e6f88d2eb.png)
- Ensure **SSL** options are **active**.

In the **Advanced tab**, insert the following content and save:

```
proxy_set_header Host $host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_max_temp_file_size 16384m;
client_max_body_size 0;

location = /.well-known/carddav {
    return 301 $scheme://$host:$server_port/remote.php/dav;
}

location = /.well-known/caldav {
    return 301 $scheme://$host:$server_port/remote.php/dav;
}
```

---
### Installing and Configuring Nextcloud
Access the configured domain, set up a username and password, and complete the initial setup.

Edit the Nextcloud configuration file `config.php` to add the following options. Here are some corrections and improvements:

```
nano volumes/nextcloud/config/config.php
```

Add the following Redis configuration at the beginning of the file:

```
'memcache.local' => '\\OC\\Memcache\\APCu',
'filelocking.enabled' => true,
'memcache.locking' => '\OC\Memcache\Redis',
'redis' => 
array (
  'host' => 'redis',
  'port' => 6379,
  'timeout' => 0.0,
  'password' => 'YOUR_REDIS_PASSWORD',
),
```
Include the following configuration at the end of the file:

```
'trashbin_retention_obligation' => '30, 60',
'overwriteprotocol' => 'https',
'maintenance' => false,
```

**Restart Containers to Apply New Configurations:**
```
docker-compose down && docker-compose up -d
```
**Verifying Nextcloud (ImageMagick)**

After verification, you may see a warning related to ImageMagick. For more information on this message, refer to the Nextcloud Docker [Issue #1414](https://github.com/nextcloud/docker/issues/1414#). This warning can generally be ignored as it does not affect the functionality of Nextcloud.

---
### Configuring OnlyOffice
First, configure a domain for **OnlyOffice** in **NPM**. The configuration is similar to NPM; just change the fields below, leaving the rest as shown in the NPM images above:

Domain for OnlyOffice
- Forward Hostname / IP: onlyoffice
- Forward Port: 80
  
Configuring **OnlyOffice** in **Nextcloud**
Access your **Nextcloud** as admin, go to Settings, and then to **OnlyOffice**.

**OnlyOffice Configuration**

- Document Editing Service Address: **Domain of OnlyOffice** configured in **NPM**
- Internal Document Editing Service Address: **Domain of OnlyOffice** configured in **NPM**
- Document Editing Service Internal Requests Address: **Domain of Nextcloud** configured in **NPM**

**OnlyOffice Document Settings**
Here are some personal settings for how OnlyOffice will behave and what files it will support:

![Captura de tela de 2021-04-05 18-29-57](https://user-images.githubusercontent.com/981368/113629619-0f5f3000-963d-11eb-8242-39be3c5b6bb1.png)

---
### Scheduling Nextcloud Cron Jobs with Systemd
Testing directly with cron was not as effective as using Systemd, which offers more features and is much lighter. Start by copying the `nextcloudCron.service` and `nextcloudCron.timer` files from the Scheduling directory to `/etc/systemd/system/`, then execute:

- (Enable the service at system boot)
`systemctl enable nextcloudcron.timer`
- (Start the service immediately)
`systemctl start nextcloudcron.timer`
- (Check the service status)
`systemctl status nextcloudcron.timer`

In the `nextcloudcron.timer` file, the variable `OnUnitActiveSec=5min` is set to update the Nextcloud cron every 5 minutes. The default value is 5 minutes. Adjust it to your preferred value.