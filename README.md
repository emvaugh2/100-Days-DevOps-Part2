# 100 Days of DevOps with KodeKloud - Days 11 - 20


Greetings! Welcome back. I'm still not exactly sure how I'm going to format this so just take it as me taking notes on what I'm doing. 


## Day 20: Configure Nginx + PHP-FPM Using Unix Sock
## Day 19: Install and Configure Web Application
## Day 18: Install and Configure DB Server
## Day 17: Install and Configure PostgreSQL
## Day 16: Install and Configure Nginx as an LBR
## Day 15: Setup SSL for Nginx
## Day 14: Linux Process Troubleshooting

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
