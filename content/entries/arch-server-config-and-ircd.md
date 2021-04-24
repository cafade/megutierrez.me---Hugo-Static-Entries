---
title: "Home Arch Server config and InspIrcd"
date: 2021-04-24T10:19:20-05:00
draft: false
---
From my Arch Linux home server I manage two of my subdomains, learnin and learninweb, which are part of the TLD 
homelinuxserver.org. These subdomains are offered for free by [FreeDNS](https://freedns.afraid.org/)
in a service which includes control over web redirections and dynamic DNS management.

Due to some limitations I will describe in the following paragraphs and to avoid the need to always type "https://" to 
get into my server, I set up a web redirect like so: [learnin.homelinuxserver.org](http://learnin.homelinuxserver.org) 
automatically redirects to https://learninweb.homelinuxserver.org.

I went this route because I think it is the perfect service to accomplish what I wanted, the most important reason is 
that by hosting websites and webservices from a residential ISP, I'm bound to have an always changing IP instead of a 
static one. To cope with this I had the freedom to use the dynamic DNS client called Ddclient, which I configured with 
my FreeDns account credentials and my desired subdomain.

One problem I faced in while setting this up, is the fact that my internet provider, like most, blocks incoming traffic 
to port 80 in my router. This in truth is instead used by the router to serve the web interface for network management. 
I can get around this by opening port 443 on the router and make use of exclusively https traffic, avoiding the need to
rely on port 80 for plain http traffic.

This in turn brought another inconvenience. which is, if I want to get free certificates from Letsencrypt to encrypt the
https traffic, I couldn't use Certbot. While reading the docs for Certbot I found no other option than to use port 80 to 
authenticate my subdomain or to go for the DNS route, but given that I'm not the owner of the TLD, this is just not 
possible, because I would need to change the DNS records from the domain registrar configuration.

I finally got on the right track by reading in the Letsencrypt docs about other supported clients different to Certbot.
The one client that ended up accomplishing the task at hand was acme.sh, a shell script shipped by Arch Linux in the
community repo. It supports the issuing of certificates by using just port 443. For example, with the following 
command arguments:
 ```
# acme.sh --issue --alpn -d learninweb.homelinuxserver.org \
--pre-hook "service nginx stop" --post-hook "service nginx start"
``` 

After issuing this command, I had a working Nginx server using my domain's https certs. Now, this Nginx server is 
configured to listen on port 80 and port 443, and to automatically force traffic through port 443. This only works for 
LAN connections due to the limitations on the router I mentioned earlier. This required I add the corresponding entry 
inside the hosts file on my other Linux machines.

On the root dir for the webserver I configured http authentication and an accompanying fail2ban jail. This is to ensure 
that only I have access to a statically generated page with a dashboard I'm planning on building to manage the various 
services I currently run. Some of these services are Nextcloud, Plex, Portainer, Syncthing, Cryptpad, Adguard Home, InspIrcd, 
and a custom whatsapp bot made with the node.js library [wa-automate](https://github.com/open-wa/wa-automate-nodejs).

On the subdirectory /irc of this Nginx server I set up a reverse proxy for the self-hosted IRC client called 
[The Lounge](https://thelounge.chat/).

## InspIrcd and Anope services

I decided to just reuse the Letsencrypt certs I issued before, for my InspIrcd. For the renewal I made the command look 
like so:

 ```
 # acme.sh --renew-all --alpn --pre-hook "service nginx stop" \
 --renew-hook "service nginx start" \
 --renew-hook "/home/learnin/.local/bin/copy-letsencrypt-cert-to-inspircd.sh"
``` 

The certificates are then copied over to the InspIrcd config directory with this [script](https://github.com/cafade/Scripts-and-Tools/blob/master/copy-letsencrypt-cert-to-inspircd.sh) 
modified to suit my needs. For now, I'm debating whether to create a separate private network config, or keep the
server as it is.

For the security configuration, I made sure that the server only accepts connections using ssl, and also set up a pair
of SQL databases which store the list users authorized to be authenticated in the irc server, and the list of which 
users are any kind of operative.

For the use of services like Nickserv, etc. I'm using Anope. I compiled it to include the openssl module. This module 
was required to connect to the InspIrcd server through a port listening exclusively to requests using ssl.

At the time of writing this, I only have configured a BotServ moderator bot and a misc [Hubot](https://hubot.github.com/) 
bot; But I'm planning on adding functionalities like a server and website monitor for all of my machines connected to this
IRCd network.

The Hubot bot is running with a module installed which allows it to execute bash scripts residing in a secure directory.
Some of these scripts are for example [yt.sh](https://github.com/cafade/Scripts-and-Tools/blob/master/Mine/yt.sh) and
[yt_link.sh](https://github.com/cafade/Scripts-and-Tools/blob/master/Mine/yt_link.sh), that run in conjunction to play
music from YouTube through a pair of USB speakers. Also, this [script](https://github.com/cafade/Scripts-and-Tools/blob/master/Mine/say) 
to play text to speech in an interactive session, and more.