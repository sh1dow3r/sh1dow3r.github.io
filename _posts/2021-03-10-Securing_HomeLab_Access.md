---
layout: post
title:  "Securing HomeLab Access"
categories: Homelab, Cloudflare, Virtualization
---


## **Introduction**

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Using only VPN as your main method to access you homelab environment is not alway as reliable as you would hope so.  VPN sometimes is unstable and hard to troubleshoot, especially remotely.
If you have wonder if there is a way to access your web console of your homelab hypervisor externally and restrict access to it, this blog is what you are seeking. In this blog, I will show a use case where you can use an open-source reverse proxy alongside Cloudflare "awesome" dashboard functionality.</span>

## What you will need?
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
For this post, you will need three things: <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
1- Router that has public IP. In this blog I choose pfSense ❤   <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
2- Virtual machine that has docker installed and has access to the internet.  <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
3- Registered domain name through Cloudflare. <br />

#  **Terminology**

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
Before diving into the project, let's take a define some terms to establish ground-level knowledge:
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/CF_dashboard.png"/>
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
`Cloudflare Dashboard`: Cloudflare dashboard is where you define the DNS records and modify them. Since we're on the subject, the dashboard so many many AMAZING services that I can't even begin to fathom what you could accomplish with them. For the time being, we will stick with the basics ones such as:
<br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> 1- DNS: To define our DNS records <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> 2- SSL/TLS: To modify TLS negotiations with the proxy and other parties. <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 1.1em;"> 3- Access: Protect internal resources by requiring authentication <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 1.1em;"> To read more about these services, visit their [documentation]( https://support.cloudflare.com/hc/en-us/articles/205075117-Understanding-the-Cloudflare-dashboard) page.
<span style="color: #f2cf4a; font-family: Babas; font-size: 1.1em;">  
`Traefik`: Traefik is a dockerized and open-source reverse proxy and load balancer typically used with microservices in the cloud(Docker swarm, Kubernetes).
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/Traefik.png"/>
</span>

## **Install/Setup**

### - Cloudflare Setup

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> 
Under [Cloudflare dashboard]( https://dash.cloudflare.com)
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> Go to the DNS tab and create a new 'A' record that correspond to you public IP. An example is shown in the screenshot below. This record will be used for the traefik web interface.
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/CF_DNS.png"/>

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;;">
After adding the 'A' record, you will need to add a 'CNAME' record for each microservice you need to access externally. The CNAME record will point back to the Traefik 'A' record "dynamic" we added earlier.
Here's an example of adding a web01 record.
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/CF_web01.png"/>

#### Securing your mircoservices with Cloudflear Access

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
Cloudflare provides you with functionality where you can limit the access of specific page to certain users. The identity provider varies based on your choosing. Every identity provider has their instruction written after you choose it.
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
After you setup your login method create an Access policy for your microservice you want to limit its access. 
In the screenshot below, I'm creating an access policy that limits the access of web01 page to the user with `l33t@gamil.com` email. There are many ways to restrict access to a page that is better than what I'm showing that Cloudflare feature such as ("Emails ends with", "IP range", "Access Service Token", ... ).
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/CF_AccessPolicy.png"/>

### - VM setup
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
After we install the dependencies namely docker and docker-compose in the VM.
Open the terminal in you VM and clone this repo:  <br />
`root$ git clone https://github.com/sh1dow3r/Traefik_CF`  <br />
`root$ cd Trafik_CF`
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
Inside the repo you will need to apply two task
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
1- Generate a certificate for Traefik microservices and place it in certs directory, which can be easily done with this command  <br />
`mkdir -p certs; openssl req -x509 -newkey rsa:4096 -nodes -out certs/cert.crt -keyout certs/cert.key -days 365`  <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
2- Make note of your Global API KEY and email from your cloudflare account. This information can be found in your under your profile [Cloudflare dashboard]( https://dash.cloudflare.com/)  <br />
After you have taking the global API Key, add it to the dockerfile in Traefik folder, and add your email as well as shown in the screenshot below:
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/CF_API.png"/>
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/Traefik_Dockerfile.png"/> 

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> 
After setting up all the global variables, we need to make small changes to the `traefik.toml` under `traefik` folder. Edits will be as follows:

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">1- Change your email under `[acme]` <br />

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">2- Change the domain to your domain under `[acme.domains]` <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> 3- Make sure to set up the right IP of traefik VM. <br />
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/Traefik_IP.png"/>
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">4- Make sure to add each mircoservice you would like to add to both `[backends]` and `[frontends]` following the same format of the existing record `web01`.

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;"> Final step would be to bring spawn the docker instance using the following command: <br />
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
`root$ docker-compose up -d`

### - pfSense Setup

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">  
Now that we configure pfSense to redirect the traffic coming on port 80 and port 443 of the public IP to be redirected to the Traefik reverse proxy. That will be quickly done through the NAT rule to allow port forwarding and through the Firewall Rules to allow incoming traffic to come in.  
</span>
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
The Firewall rules would look like something like this:
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/FirewallRule.png"/> 
<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
The NAT rules would look like something like this:
<img src="https://raw.githubusercontent.com/sh1dow3r/layer0/gh-pages/_posts/img/Remote_Access_Homelab/NATRule.png"/> 

## Conclusion

<span style="color: #f2cf4a; font-family: Babas; font-size: 0.9em;">
 In this blog I explained how to add a secondary access to your homelab using Cloudflare free features and using Traefik reverse proxy. I also touched a bit how to configure the routes on pfSense to allow the traffic through using NAT rules. Using such method can help if you lose you VPN access to your environment and help prevent single point of failure on certain cases. </span>

# References


[Cloudflare docs](https://support.cloudflare.com/hc/en-us/articles/205075117-Understanding-the-Cloudflare-dashboard)

[Cloudflare Settings for Traefik Docker](smarthomebeginner.com/cloudflare-settings-for-traefik-docker/)

[Evan's Github](https://github.com/egallis31/traefik-elk-grafana)
