# 100 Days of DevOps with KodeKloud - Days 11 - 20


Greetings! Welcome back. I'm still not exactly sure how I'm going to format this so just take it as me taking notes on what I'm doing. 


## Day 20: Configure Nginx + PHP-FPM Using Unix Sock
## Day 19: Install and Configure Web Application
## Day 18: Install and Configure DB Server
## Day 17: Install and Configure PostgreSQL
## Day 16: Install and Configure Nginx as an LBR

Okay so while I know my way around a load balancer (LB or LBR) pretty okay, I've never configured NGINX as an LB so I'll google this as well. So, here are the tasks: configure the LBR server with NGINX, use HTTP as the LB protocol and all App Servers should be utilized, keep the Apache port that's being used on the app servers the same and make sure the Apache service is running, and lastly, do a curl test on the LBR server on port 80 to verify configuration. 

Lets go and install NGINX on the LBR using `sudo dnf install -y nginx` and get it running using `sudo systemctl enable --now nginx`. It was already installed but the service was stopped. I went and got it started. Now, lets get the HTTP context configured. I'm not used to this so I'll look around first and see if I can get it figured out myself. The lab says the configurations are located in the nginx.conf file. Not surprised. Nothing jumps out at me right now besides the server_name but I would assume this is the load balancer's name. Going to google it. None of these google searches were very clear but I did find an article on Medium that says I need to add the IP addresses to the server_name list. This may be incorrect. I found another website that says I need to configure an upstream block. Lets try that. I guess my thing is, I don't think we're on a domain. I did `hostnamectl` and I'm not seeing anything. A quick google search says use hostname -d or -f to see your domain and I don't see anything. So I'm going to leave server_name blank as it is and just put the upstream block information in the config file. I'll use ping to the get the IP addresses of the 3 App servers. 

Once I got the IPs, I added them in the upstream backend block I created. You can force the backend nodes to listen on a specific port by specifying the socket (IP:port). Before I did that, I logged into all 3 servers and made sure Apache was running on them. They were all running on port 8084. I went back to the LBR server and restarted NGINX. Going to perform the curl test from another machine to see if everything works. I tried to log into the DB server since it's on the internal network. It didn't recognize the hostname. I stayed on the jump-host which is in the DMZ and did the test `curl http://stlb01:80` and received the usual output. I should've done a before and after now that I think about it. Going to submit my answers now. 

I did not pass. Lets see what the issue was. I'll use AI next time. Taking a break. 


## Day 15: Setup SSL for Nginx

I actually have no clue how to do this so I will be using Google and AI. I haven't configured many servers outside of Cisco and Solarwinds products. 

So we need to install NGINX on App Server 2, move the self-signed SSL certificate to "some appropriate location and deploy the same in NGINX" (whatever that means). Then create an index.html file to display `Welcome!` under the NGINX root. Lastly, do a curl test to the stapp02. I used `sudo systemctl status nginx` to see if it started and was enabled. So far so good. 

So the SSL certificate is located in the `/tmp/` directory. I'm going to google where this needs to go in the NGINX configurations. I found a website called cert-depot.com which outlines this process. So first, I need to make the directory (and parent directories) for `/etc/nginx/ssl`. I need to move the certificate and key under the ssl directory. Then change the permissions on the key. Lastly, I need to make the root user the owner of the files. 

First, I'll install and enable --now NGINX. 

(Troubleshooting. I ran into an issue with NGINX failing to start. I couldn't find out what the issue was by reading the logs in the systemctl status output. Once I put that information in Google, it gave me a bunch of solutions and told me to check line 58 in the file. I went to line 58 and realized I had an indentation error which isn't truly accurate. I just forgot to comment out the "If this is a TLS enabled server," part which should not have been readable by the file. Once I tried to restart NGINX again, I got a different error code. Now it's saying it cant load the certificate key. I'm assuming this is an ownership issue. I googled it and that looks like the issue. I would assume nginx needs ownership over the key. 

Before changing ownership, I made sure the NGINX user existed on my machine using `cat /etc/passwd | grep nginx` and saw the nginx owner. I used `sudo chown nginx:nginx /etc/nginx/ssl/nautilus.key` to change the ownership. That didn't fix the issue. I did `sudo nginx -t` to test and it's saying this file doesn't exist. Maybe it's a typo somewhere. 

Sure enough there was a typo. It's been a long work day and I didn't realize I left the last apostrophe off of the ssl_certificate_key line. I found this by myself so AI didn't solve this for me. 

Once I fixed that, I restarted NGINX and it worked. I did a `curl -Ik https://stapp02` test which gave me a HTTP 200 response (I think that's the proper response). Then I did another curl test to the localhost to see my Welcome! output. I actually did ask AI to generate the new index.html file because while I have written some HTML in the past, I don't know how to do it anymore now. So I copied and pasted that. I then received a green success check for the lab. 

A few takeaways, Copilot on Bing is not great. Google AI is better for a conversation. Also, here's that website I found about the SSL certificate and key: https://cert-depot.com/guides/self-signed-cert-nginx

## Day 14: Linux Process Troubleshooting

The issue is the Apache service is malfunctioning on one of the application servers. We also want Apache to run on port 8082 on all services. I'm going to do a `systemctl status httpd.service` to investigate. On stapp01, it's saying httpd.service failed to start because it couldn't bind to the socket 0.0.0.0:8082. Using `ss -tunlpe`, I see that the sendmail.service is already on that port so I'm going to shut down that service and restart Apache. I also enabled Apache and did a curl on the localhost:8082 and received output. Looks like it's working so far. On to stapp02.

On stapp02, I went through the same troubleshooting process. Apache was already listening on socket *:8082 which I believe is for both IPv4 and IPv6. I did a `sudo systemctl status httpd.service` and the service was working properly. I just enabled it just for persistence. I also received the same output with the curl test. Lets move to stapp03.

It was the same exact situation on stapp03 as it was on stapp02 so I followed the same process. After that was completed, I made sure I did a few more curl tests just to make sure routing was in place. On stapp02, I did `curl http://stapp02:8082` and to stapp01. I received the expected output. Then, I logged back into stapp01 and did curl tests to stapp02 and stapp03 and received the proper output. 

Finally, I had KodeKloud check my work and received a green check for success! Pretty simple. I've done a lot of this work while studying for the RHCSA so this adds up. 

## Day 13: IPtables Installation And Configuration

The goal is to install iptables on each application server, block port 3001 for everyone except the LBR host (I'm assuming this is the load balancer), and make it persistent.

Okay first, let me install iptables. I'll use `sudo dnf whatprovides iptables` and install the package. That revealed a package called iptables-nft. I installed it on all 3 servers. Now, lets see how to list the current table rules and how to create a rule specifically for the load balancer server. The Unix & Linux website says use `iptables -L -v` to see the current rules. They're empty. I'm going to grab the IP address of the LB server and try to make deny rule for all traffic besides the LB. How I'm thinking about this is since the policy is Accept, there's an implicit allow at the end of these iptables. So all traffic should be allowed. I know with Cisco firewalls, once you match a rule, you won't be able to hit any rules below that. So if I deny traffic for everything except the LB on port 3001, if anybody tries to reach port 3001, they'll be blocked and will not move onto the implicit allow rule at the bottom of the iptable. So even if all the traffic is being allowed, it will still be dropped for this port. Just my rationale. 

UPDATE:

Alright after thinking this through with AI, I decided to create an allow rule for the Load Balancer and a deny rule for everybody else on port 3001. I made a rules for both TCP and UDP so that's 4 rules in total. 

Now, this lab was a bit of a nightmare because the iptables commands weren't very clear. I also wasn't sure if the LBR host meant load balancer. Lastly, I had to run these commands on all 3 application servers and do verifications on non-LB servers and the LB server to make sure the rules work. Here's how I went about it.

I had to install both `iptables` and `iptables-services`. Then, we `enabled --now` the iptables service to make sure that it starts now and persists across reboots. Then, we had to place the rules in a speciic spot above the reject rule at the end of the INPUT chain (for inbound or ingress traffic where the LB would be coming from). I never made an iptable rule before (I've only worked with `firewall-cmd` before) so the format was something like: `sudo iptables -I INPUT 5 -p tcp -s 10.X.X.X --dport 3001 -j ACCEPT`. So nothing crazy but a few things: I didn't see any great examples in the man pages so I didn't know how to create this rule and I'm also used to making rules in the Cisco FMC or ADSM instead of doing it from scratch in the CLI. So this is a bit new to me doing this on the command line. It all makes sense though. The -I flag is which chain and where on the chain to put the rule. -p is for protocol so choose TCP or UDP generally speaking. -s is the source IP. --dport is the destination port the source IP is trying to get to. -j flag is for allow or deny (or return). Now, why the letter j? I have no cluse. 

So, I made those 4 rules and then verified they were in the iptables using `sudo iptables -L INPUT -n --line-numbers`. Now, to save this was even more weird. I had to use the command `sudo sh -c 'iptables-save > /etc/sysconfig/iptables'` which, I don't even know what sh -c does. I got this straight from AI. You can grep on that file to see the changes you made. To do a quick verification check for persistence without a reboot, you can just restart the iptables service `sudo systemctl restart iptables`. Then check to see if the rules are still there. 

Now, I had to do a netcat test `nc -zv <server> 3001` from the non-allowed server and the LB server. I got a timeout for the non-allowed server and a connection from the LB server. I got the LB server's IP by pinging it froma different server. I had to repeat all of this on the other 2 servers. I also used a `telnet <server> 3001` test where I got a connection or no connection at all. Once everything was verified, I got a successful message for the lab. 

SHEESH. Learned a lot. Humbling considering my background. 


## Day 12: Linux Network Services

As a network engineer, this one was a little embarrassing. We had to troubleshoot the Apache HTTPD service on one of the 3 servers. I'm glad they gave us three servers because the working two servers allowed us to see what the solution should look like. We needed to make sure the Apache server was listening on the correct port and that we could curl to the server from a different server. I was randomly assigned port 8085. I used `ss -tunlpe` to see what services were listening on which ports. There was a servicemail process running on that port so I stopped the service and restarted my Apache service. In the event logs, it showed that HTTPD was now working. Once I curled to the server from a working server, I received the error "route not found". 

Beofre we checked the neworking portion further, I did a `curl localhost:8085` on the broken server to make sure the output was the same as the working servers which ended up working properly. So curl wasn't the issue here. I compared the `ip route show` and `ip addr show` output from the working servers to the non-working server. The output was the same albeit the IP addresses were different. I pinged from a working server to a broken server and the pings came back successfully so the route had to be in place. 

This is where I used AI to help me troubleshoot. I was told to check the firewall but `firewalld` wasn't even installed to use `firewall-cmd` commands. AI had me use the iptables to troubleshoot. The default policy was allow and we didn't see anything specifically stating to block the traffic. AI gave me a rule to implement and once I added the rule, I was able to curl from the working server to the broken server. Now, I'm not familiar with iptables so this would've stumped me. Nonetheless, it was a good learning experience!


## Day 11: Install and Configure Tomcat Server

I'll take to look back over this one. Installing the Tomcat server was pretty straightforward but they wanted me to configure it in a specific way. I had to google that part as I wasn't familiar with this type of server. 
