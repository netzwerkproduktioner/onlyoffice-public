# ONLYOFFICE Server
Far-reaching automation for installation Onlyoffice with a single cloud-init script.  

The main file is the __cloud-config.yml__, a cloud-init file, which is tested on cloud-systems from Hetzner. 
Please consult https://canonical-cloud-init.readthedocs-hosted.com/en/latest/index.html if you're not sure what a cloud-init file does.  

# What does this file do?  
On a Debian 10 system the file creates config files and runs a setup which finally gives you an __Debian 10-Server, running NGINX, which serves you an OnlyOffice instance running in a docker container__.  

This server configurated to get accessed by a second server/system to use OnlyOffice within.
I.e. you can simply connect this server with a Nextcloud instance. 

# Installation time  
Depending on the power of your machine the whole installation process will need up to 20 min.  


# Before you start with cloud-config.yml  

1. Set the __username__ in the _users_ section and in _ssh config-section_.

2. Replace the example __authorized key__ with your(s) in the _users_ section.

3. Replace the __subdomain + domain__ (i.e. sub1.your-domain.tld) in the whole file.  

4. Replace __CONTACT_EMAIL__ below "/etc/dehydrated/config" in the _write\_files_ section.  

5. Replace the (sub)domain entry below "/var/lib/acme/domains.txt" with __your (sub)domain__.  

6. Set a secret (a __JWT-Token__/String) and replace it at the __placeholder {{ your secret }}__.  

7. __Replace the URLs for your custom HTML__ in the \*\*\*CUSTOM HTML*** section.  


# Run cloud-config  
Run the config by copying the whole __cloud-config.yml__ (./data/configs/hetzner) into the "data" field in Hetzners server environment (creating a new server) or use the Hetzner API instead (not scope of this readme).  

Other provides may have different workflows. Check if your setup supports cloud-init procedures. Examples can be found here: https://cloudinit.readthedocs.io/en/latest/topics/examples.html  


## files created during cloudinit automatically  

### config files for nginx  
* override of _/sites-available/default_  
Overrides the default file to redirect everything, thats not https to https. Also handles the acme-challenge.  

* creates a _config file for subdomain_  
Creates a server config file for onlyoffice on specific (sub)domain.  

### adds entry to crontab  
The renewal of SSL-certificates by dehydrated is added to crontab.  
The init script appends to /etc/crontab.  

### config files for dehydrated  
* creates _/etc/dehydrated/config_
This ini file overrides default param settings of dehydrated.  
You will need an email-address using letsencrypt.  
* Params:  

      CA="letsencrypt"  
      BASEDIR="/var/lib/acme"  
      WELLKNOWN="/var/www/acme-challenge"  
      CONTACT_EMAIL="certs@your-domain.tld"  

* creates _/var/lib/acme/domains.txt_  
This text file contains all domains, which you wanted certificates for.  

#### structure for dehydrated  

* config folder  
/etc/dehydrated/config

* folder for the letsencrypt challenge  
/var/www/acme-challenge  

* folder for certs, domain.txt, ..  
/var/lib/acme

* certificates  
Params:  

        ssl_certificate /var/lib/acme/certs/${your-domain}/fullchain.pem;  
        ssl_certificate_key /var/lib/acme/certs/${your-domain}/privkey.pem;  
        ssl_trusted_certificate /var/lib/acme/certs/${your-domain}/chain.pem;  

### docker-compose file for onlyoffice  
* creates _/opt/onlyoffice/docker-compose.yml_  
This file controls your onlyoffice docker setup.  

## files getting modified during cloudinit  

### fail2ban  
* modifies settings in _/etc/fail2ban/jail.local_  
sets params for "enabled", "banaction", "bantime", "maxretry". This ini file only contains additional settings or overrides for defaults.  

### sshd  
* modifies settings in _/etc/ssh/sshd_config_  
sets hardening settings in ssh (i.e. SSH-Key-Authentication, Ports, ..)  

### Further  
* get static HTML files from your (public) repo.  

* generate symlink(s) for sub-config files in nginx
For every config file in your ../sites-available/ folder you have to create a symlink in your ../sites-enabled/ folder.  

        ln -s /path/to/sites-available/file /path/to/sites-enabled/file

## SSL Certificates (with dehydrated) 
* run dehydrated script  
__Be aware your mail address is already registered__. Otherwise you run into an interactive dialog, which will break your automatic workflow.  

        ./dehydrated --register --accept-terms
        ./dehydrated --cron

* run your docker-compose.yml

        cd /opt/onlyoffice/ 
        docker-compose up -d  


# Manual SetUp

## DNS settings  
Set IPv4 and IPv6 your DNS for all your subdomains you want to register in _domains.txt_.  


# Updates/Upgrades  

* delete docker volumes  
Delete or archive your docker volumes __except your custom html folder__ before upgrading your Onlyoffice image/container. 


# Troubleshooting  
check if nginx is running -> sudo service nginx status  
ggfs. sudo service nginx reload && sudo service nginx restart  


## Check error logs  
* r3.o.lencr.org could not be resolved (5: Operation refused) while requesting certificate status  
__Check:__ Set a new address for resolver in your config and check if the error is gone.  

* SSL_ERROR_RX_RECORD_TOO_LONG  
__Check:__ Check if all your domains resolve properly  

* *43 SSL_do_handshake() failed (SSL: error:141CF06C:SSL routines:tls_parse_ctos_key_share:bad key share) while SSL handshaking  
__Check:__ supposed to be a problem on client side, @see: https://stackoverflow.com/questions/65854933/nginx-ssl-error141cf06cssl-routinestls-parse-ctos-key-sharebad-key-share/67424645#67424645  
