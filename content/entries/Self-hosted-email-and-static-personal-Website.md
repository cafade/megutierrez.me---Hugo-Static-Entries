---
title: "Self-hosted email and static personal Website"
date: 2021-04-19T21:15:50-05:00
draft: false
---
For this very website and my self-hosted email, I'm currently using a couple of ubuntu server VMs hosted in
Oracle Cloud. I used to use GCP for small personal projects, but the always free tier that oracle offers is way better 
than the one from its competitors.

As far as I know It includes 1 TB of outbound traffic per month, free usage of two of the weakest configuration 
machines, and plenty of HDD storage. These two little VMs have 1 GB of ram, and a modest CPU, which for now 
this seems to be more than enough to smoothly run what I need. Although, if the response times get too bad, I can just
migrate the website to Cloudflare's free proxy.

I configured an OpenVpn server and a Nginx server in the first VM. Then in the second one I'm running a Nginx
reverse proxy pointing to the first VM, as well as the popular stack to host SMTP/IMAP-POP3 mail, composed of Postfix
, Dovecot, and Spam assassin.

### Initial Configuration

First I set up the Oracle's virtual network to have the reserved public Ip assigned to my email VM and opened the relevant
SMTP, POP3, IMAP, OpenVpn, and Https ports. I then configured my Namecheap domain to point to this Ip in my CNAME and 
Mx records.

Having configured my DNS, I modified the default iptables rules set for Ubuntu server and set these changes be permanent.
This now allows my Vm to receive and send traffic through the relevant ports. After installing Certbot and getting 
Letsencrypt certs, I was done with all the steps required and to proceed and set up the email.


### Self-hosted Email

For this I used [this](https://www.linode.com/docs/guides/email-with-postfix-dovecot-and-mysql/) guide, which at the
time I tried to set things up, was somewhat outdated or wrong in some steps.

- The first hiccup I encountered was that the version of MySql used in the guide used the deprecated function ENCRYPT(), 
which found I had to replace with ```SHA2()```, as per [this](https://www.linode.com/community/questions/19393/encrypt-function-not-working-following-linode-process-for-mailserver-setup)
post. This also didn't work. Now, the plaintext I would be inputting into the database was never going to be hashed, 
so I had to use the Dovecot command line tool called ```doveadm``` like this:
  
```$ doveadm pw -s SHA512-CRYPT```

After this ordeal, I had my pass hashed using the sha512 algorithm. This also means that I need to feed this hash manually 
to the database, which I guess is not ideal. I'll have to look up later how to correct this.

- The second problem I had was in the section of the guide in which the mail directory for our mailuser is created. Maybe 
I misunderstood what it said, but the command:
  
```# mail -f /var/mail/vhosts/example.com/email1``` 

with the flag _-f_, created a directory called ```<example.com>``` with a file in it called ```email```. 
<br><br>This is completely wrong because in the configuration file for Dovecot called ```conf.d/10-mail.conf``` we had to make sure 
to tell Dovecot that the mail was going stored in a directory, not in a file. This is the line to set the mail location: 

```mail_location = maildir:/var/mail/vhosts/%d/%n/```

I fixed this inconsistency by creating a folder like:
```/var/mail/vhosts/example.com/email1/``` 
Then creating a folder inside it called _tmp_: 
```/var/mail/vhosts/example.com/email1/tmp```
  
- Everything else in the guide worked without any problem. I then enabled all the systemd services provided and started 
them. Also had to configure fail2ban jails to deal with bots looking for open proxies, vulnerable login forms, etc.

For the SMTP part of the stack, I needed to improve my domain's reputation to avoid being trapped in spam folders, so 
I looked up Oracle's documentation about setting up a reverse DNS entry for my reserved ip.

In the docs they said you need to send a ticket to your cloud 
account admin for them to allow this change. Because I am the admin of my own cloud account I needed to create an admin
account in Oracle's support platform and then I would be automatically granted the privileges to authorize the reverse DNS or PTR.

Well this didn't happen. At the moment of creating the account in the support platform I never got these so called
admin rights, even thought I linked the root/admin email and account to the support one. To change your cloud account 
administrator, or to even approve new accounts in the support platform linked to the cloud account, you need to already
be an admin of the support part of your cloud.

Stuck in this catch 22, I gave up for now and looked for an SMTP manager provider.

I ended up picking SendGrid to send and reply to email through their web mail API, while all incoming mail is directed 
to my own server. I'll also need to revisit this to make things the right way.

### Static Website

For the site I used a simple template offered by Html5up and mangled it a bit to look a bit more to my liking. Decided
to keep a low profile on the resource usage and foolishly tried to implement my own way of serving the content in 
spanish and english depending on the user's browser language and preference, and, with only vanilla Js.

I then read about static page generators like Jekyll and Hugo, which seemed like a much better alternative than what I
just did. Right now this entry has been generated using Hugo, with a template called PaperMod. I still need to migrate
the messy work I did before to a Hugo template, which by the way, the original Html5up template I used has also been
ported to Hugo [here](https://themes.gohugo.io/hugo-identity-theme/).

For now, I'm just going to add a subdirectory inside Nginx's configuration which points to the root dir of the Hugo content,
in which I push to with rsync.

> TODO: fix gz error in nginx
> 
> TODO: enable automatic password hashing for the virtual users table
> 
> TODO: migrate all site content to hugo
##### *_[test results permalink for the website ssl config and in general](https://check-your-website.server-daten.de/?i=09aab5be-7c5f-4a27-a1fa-bce2311c25ab)_