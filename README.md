# New to Apple's account-driven enrollment concepts?

Apple's account-driven enrollment is a streamlined process that simplifies the setup and management of Apple devices that are already in production or not eligible to Apple's Automated Device Enrollment (ADE) program. 

By utilizing Managed Apple IDs (or Accounts as Apple call them now), organizations can automate the enrollment of devices into their mobile device management (MDM) system. This process can be either used to enroll institutional devices (Account-driven DEVICE enrollment or ADDE) or to enroll personally owned devices (Account-driven USER enrollment or ADUE). 

As of today, Apple is heavily pushing towards ADUE for any Bring Your Own Device (BYOD) enrollment workflows.

Here’s why you should consider implementing account-driven enrollment:

 🔒 Security - End-users are required to enter their own credentials instead of relying on a generic enrollment account.
 
 😶‍🌫️ Privacy - Account-driven enrollment unlocks cryptographic separation of personal and professional data.
 
 🚀 Accessibility - Provide the best experience with a streamlined authentication process.
 
 ⚙️ Compatibility - Vision Pro is landing soon in Europe, and today it can only be managed through account-driven enrollment. And more generally Apple is pushing towards ADUE for BYOD.
 

Here are the components required to setup account-driven enrollment:

 - An MDM solution that supports account-driven enrollment
 - An Identity Provider (optional but recommended)
 - An Apple Business Manager or Apple School Manager tenant
 - Managed Apple IDs (or Accounts as Apple call them now)
 - A domain name that you own
 - A Service Discovery server that’ll return your MDM solution URL

Find more information on how to set up account-driven enrollments using Jamf's Documentation Center:

 - [Enrollment with Jamf Pro](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/Enrollment_into_Jamf_MDM.html)
 - [Prepare for Account-Driven Enrollment with Managed Apple IDs and Service Discovery](https://learn.jamf.com/en-US/bundle/technical-articles/page/Prepare_for_Account-Driven_Enrollment_with_Managed_Apple_IDs_and_Service_Discovery.html)
 - [Account-Driven User Enrollment Experience](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/Account-Driven_User_Enrollment_Experience_for_Personally_Owned_Mobile_Devices.html)

# Apple Service Discovery Server Setup Guide

This guide will help you build a Service Discovery Server for account-driven enrollment workflows using Apache and the files shared in this GitHub project.

This guide is a proof of concept to showcase how you could enroll Apple devices either in ADDE or ADUE depending on their device type (Mac, iPhone, iPad, RealityDevice) by tweaking your Service Discovery server. You should adapt it to your own environment.

More information on Apple Service Discovery servers [here](https://developer.apple.com/documentation/devicemanagement/discover_authentication_servers)

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
