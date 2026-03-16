---
title: Cobalt Strike Redirectors using Apache-SSL
category: Red-Team
excerpt: This is how you setup single or multiple redirectors for Cobalt-Strike using malleable-c2 profiles and Apache SSL
header:
  overlay_color: "#000"
  overlay_filter: "0.75"
  overlay_image: /assets/images/sample/homelab/cs.webp
  teaser: /assets/images/sample/homelab/cs.webp
tags: HomeLab IT RED-Team
toc: true
classes: wide
comments: true
---

# Overview

Redirectors are systems designed to proxy traffic from a target network to private attack infrastructure. They play a vital role in ensuring operational security and serve several key purposes:  

#### **1. Preventing Unauthorized Access**  
Redirectors act as a shield, preventing unauthorized parties from accessing the underlying attack infrastructure. They can:  
Only allow authorized users or systems to connect to the actual infrastructure.  
Make the infrastructure appear harmless or unrelated to its real purpose, deterring unwanted attention.  

#### **2. Shaping How the Infrastructure Appears**  
Redirectors control how the target perceives the infrastructure by:  
Modifying content, headers, or other elements to make traffic appear legitimate and unsuspicious.  
Mimicking well-known platforms or architectures to blend in and avoid detection.  

#### **3. Safeguarding the Attack Chain**  
By acting as a buffer, redirectors help ensure that the overall attack chain remains intact, even if a part of it is exposed. They achieve this by:  
Keeping different parts of the infrastructure separate to minimize the risk of exposure.  
Hiding communication patterns to make it harder to trace or identify the destination.  
Providing a way to swiftly redirect traffic to backup systems if a redirector is compromised.  

In red team operations, penetration testing, or other offensive security scenarios, redirectors are essential. They offer a blend of protection, control, and flexibility, making them a cornerstone of effective attack frameworks.

### **4. Cobalt Strike Redirectors using Apache-SSL**


## Step -- 01

On the cobalt strike team server, we've to generate two files. `public.crt` and `private.key`.

    root@teamserver: openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out public.crt -keyout private.key
{: .notice--info}

Once the command is executed,  you will be presented with the below prompts. 

```
Country Name (2 letter code) [AU]: 
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]: 
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []: 172.16.100.3
Email Address []:
```


Here the common name can be the domain you own. for my case I'm just using the ip of my teamserver. 

![1.png](/assets/images/sample/Cobaltstrike/1.png)

now copy both the files from your teamsever to the redirector machines `/etc/apache2/ssl` 
by creating an ssl folder.

## Step -- 02

`root@redirector1: sudo apt install apache2`
{: .notice--info}
`root@redirector1: sudo a2enmod ssl rewrite proxy proxy_http`
{: .notice--info}
`root@redirector1:~# mkdir -p /etc/apache2/ssl`
{: .notice--info}
![2.png](/assets/images/sample/Cobaltstrike/2.png)

`root@redirector1: sudo nano /etc/apache2/sites-available/default-ssl.conf`
{: .notice--info}

![3.png](/assets/images/sample/Cobaltstrike/3.png)

 Example of default-ssl.conf
 
```
<IfModule mod_ssl.c>
    <VirtualHost *:443>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        
        SSLEngine on
        SSLProxyEngine on
        SSLProxyCheckPeerCN off
        SSLCertificateFile /etc/apache2/ssl/Your-Team-server.crt
        SSLCertificateKeyFile /etc/apache2/ssl/Your-Team-server.key
        
        <Directory /var/www/html>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
        </Directory>
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
</IfModule>
```

Make sure to have these in the default-ssl.conf
`SSLEngine on`
`SSLProxyEngine on`
`SSLProxyCheckPeerCN off`

`root@redirector1: sudo systemctl restart apache2`
{: .notice--info}

Now open a browser and open https://redirector1. you should see apache now running with `SSL`.

![4.png](/assets/images/sample/Cobaltstrike/4.png)

Quick thing to check in here if you've the right certificate is that by checking the cert. your should see your `team-server` ip or the `FQDN` in the certificate.

![5.png](/assets/images/sample/Cobaltstrike/5.png)

## Step -- 03

Now for the team-sever to communicate with our apache ssl. As `cobalt-strike` is written in java, the public certificate and private key need to be imported into a Java KeyStore.

First, we need to combine the separate public and private files into a single PKCS12 file.  Set the password to anything you want, but remember it.

`root@teamserevr: openssl pkcs12 -inkey private.key -in public.crt -export -out tserver.pkcs12`
{: .notice--info}

You will be prompter to enter password:

```
Enter Export Password:
Verifying - Enter Export Password:
```

Do enter the password you can remember. Once done, you'll see a file called `tserver.pkcs12`

That PKCS12 file can then be converted to a Java KeyStore using the `keytool` utility.

`root@teamserevr: keytool -importkeystore -srckeystore tserver.pkcs12 -srcstoretype pkcs12 -destkeystore tserver.store`
{: .notice--info}

You'll be prompted asking to enter the password. You can use the same password as before. make sure you remember it as we'll be using it in the `Malleable-C2-Profiles`. 

The password of this new store can be the same or different.  At this point, the `.pkcs12` file can be deleted.  The new Java KeyStore needs to be referenced in a Malleable C2 profile before it can be used.

https-certificate {
     set keystore "tserver.store";
     set password "password";
}


The Cobalt Strike Team Server expects the KeyStore to be in the same directory as itself.  Launch the Team Server with the updated profile.

`root@teamserver: ./teamserver IP-address passwword@123 apt.profile`
{: .notice--info}