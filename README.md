# Apple JSON Discovery Server Setup Guide

This guide will help you prepare a Linux machine to set up a JSON Discovery Server for account-driven enrollment workflows using Apache and the files from this [GitHub project](https://github.com/vbnin/Apple-JSON-discovery-server).

This guide is a proof of concept to showcase how Apple devices can be enrolled using the account-driven enrollment method. You should adapt it to your own environment.

## Prerequisites

Before you start, ensure you have the following:

- A machine running Ubuntu or Debian (No machine? Check out additional information at the bottom of this guide!)
- Sudo privileges on the machine
- Access to the internet
- A registered domain name

If it hasn't already been done, create a DNS redirection from your registered domain name to your machine public IP address. This step may be different depending on your networking environment.



## Step 1: Prepare the OS

Download the latest package lists from the repositories:

```
sudo apt update -y
```


## Step 2: Install Required Packages

Install Apache, Git, Certbot, and the Certbot Apache plugin:

```
sudo apt install -y apache2 git certbot python3-certbot-apache
```

Optional - If a firewall is currently active on your machine, allow HTTP and HTTPS connections for Apache using the UFW command:

```
sudo ufw allow 'Apache Full'
```


## Step 3: Enable Apache Modules

Enable the necessary Apache modules:

```
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo a2enmod rewrite
```


## Step 4: Clone the GitHub Repository

Clone the GitHub repository to a temporary directory:

```
git clone https://github.com/vbnin/Apple-JSON-discovery-server /tmp/example-config
```


## Step 5: Move Files to Their Respective Locations

Move the `default-ssl.conf` file and JSON files to their appropriate locations:

```
sudo mv /tmp/example-config/default-ssl.conf /etc/apache2/sites-available/
sudo mv /tmp/example-config/json_files /var/www/html/
```


## Step 6: Edit `default-ssl.conf`

Edit the `default-ssl.conf` file to add your domain name. Open the file in a text editor:

```
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Find the `ServerName` directive and update it with your domain name:

```
ServerName your-domain.com
```

Review and modify if needed the rewrite rules (lines 31 to 47) configured to enable either account-driven device enrollment (ADDE) or account-driven user enrollment (ADUE) depending on the device's model family included as a query parameter. More details on query parameters [here](https://developer.apple.com/documentation/devicemanagement/discover_authentication_servers).

Example below for the "iPhone" model family, configured to follow the ADUE method through the byod.json file:

```
RewriteCond %{QUERY_STRING} model-family=iPhone
RewriteRule ^/.well-known/com.apple.remotemanagement$ /var/www/html/json_files/byod.json [L]
```

Save and close the file.


## Step 7: Edit JSON files

Edit both `adde.json` and `byod.json` files to add your MDM server URL. Open the file in a text editor:

```
sudo nano /var/www/html/json_files/adde.json
```

Find the `BaseURL` key and update it with your MDM URL:

```
"BaseURL": "https://your_mdm_url.com/servicediscoveryenrollment/v1/userenroll"
```

Save and close the file. Repeat this step for the `byod.json` file.


## Step 8: Enable SSL with Certbot

Run Certbot to obtain an SSL certificate for your domain:

```
sudo certbot --apache
```

Follow the prompts to complete the setup.


## Step 9: Test the Configuration

Restart Apache to apply all changes:

```
sudo systemctl restart apache2
```

Test access to your webserver from your local computer using curl to verify that the setup is correct and the JSON files are accessible:

```
curl -I https://your-domain.com/.well-known/com.apple.remotemanagement
```

You should obtain an HTTP 200 code and the content-type used by the server should be application/json.
In case of SSL certificate error, you may need to fully restart your server.

Verify that you obtain different data depending on the device type specified as a parameter:

```
curl https://your-domain.com/.well-known/com.apple.remotemanagement?model-family=Mac
```
```
curl https://your-domain.com/.well-known/com.apple.remotemanagement?model-family=iPhone
```

You should obtain different JSON data using either ADDE or BYOD depending on the device type specified in the URL.

## Closing Remarks

You have successfully set up the Apache web server and configured it to serve the Apple JSON Discovery Server files. If you encounter any issues, consult the Apache and Certbot documentation for troubleshooting assistance.

**Looking for a free, cloud-hosted Linux VM to host this JSON discovery server?** This configuration has been tested on a Google Cloud Free Tier e2-micro instance. 

 - [Google Cloud Free Tier offer](https://cloud.google.com/free/docs/free-cloud-features#compute).
 - [How to run a basic Apache server using a Google Cloud VM](https://cloud.google.com/compute/docs/tutorials/basic-webserver-apache)
 - [Configure static external IP addresses for Google Cloud VMs](https://cloud.google.com/compute/docs/ip-addresses/configure-static-external-ip-address#configure)
