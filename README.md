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

(we'll come back to this one)

iptables-apply(8), iptables-save(8)


## Day 12: Linux Network Services

As a network engineer, this one was a little embarrassing. We had to troubleshoot the Apache HTTPD service on one of the 3 servers. I'm glad they gave us three servers because the working two servers allowed us to see what the solution should look like. We needed to make sure the Apache server was listening on the correct port and that we could curl to the server from a different server. I was randomly assigned port 8085. I used `ss -tunlpe` to see what services were listening on which ports. There was a servicemail process running on that port so I stopped the service and restarted my Apache service. In the event logs, it showed that HTTPD was now working. Once I curled to the server from a working server, I received the error "route not found". 

Beofre we checked the neworking portion further, I did a `curl localhost:8085` on the broken server to make sure the output was the same as the working servers which ended up working properly. So curl wasn't the issue here. I compared the `ip route show` and `ip addr show` output from the working servers to the non-working server. The output was the same albeit the IP addresses were different. I pinged from a working server to a broken server and the pings came back successfully so the route had to be in place. 

This is where I used AI to help me troubleshoot. I was told to check the firewall but `firewalld` wasn't even installed to use `firewall-cmd` commands. AI had me use the iptables to troubleshoot. The default policy was allow and we didn't see anything specifically stating to block the traffic. AI gave me a rule to implement and once I added the rule, I was able to curl from the working server to the broken server. Now, I'm not familiar with iptables so this would've stumped me. Nonetheless, it was a good learning experience!


## Day 11: Install and Configure Tomcat Server

I'll take to look back over this one. Installing the Tomcat server was pretty straightforward but they wanted me to configure it in a specific way. I had to google that part as I wasn't familiar with this type of server. 
