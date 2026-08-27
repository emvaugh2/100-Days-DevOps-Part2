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



## Day 12: Linux Network Services

As a network engineer, this one was a little embarrassing. We had to troubleshoot the Apache HTTPD service on one of the 3 servers. I'm glad they gave us three servers because the working two servers allowed us to see what the solution should look like. We needed to make sure the Apache server was listening on the correct port and that we could curl to the server from a different server. I was randomly assigned port 8085. I used `ss -tunlpe` to see what services were listening on which ports. There was a servicemail process running on that port so I stopped the service and restarted my Apache service. In the event logs, it showed that HTTPD was now working. Once I curled to the server from a working server, I received the error "route not found". 

Beofre we checked the neworking portion further, I did a `curl localhost:8085` on the broken server to make sure the output was the same as the working servers which ended up working properly. So curl wasn't the issue here. I compared the `ip route show` and `ip addr show` output from the working servers to the non-working server. The output was the same albeit the IP addresses were different. I pinged from a working server to a broken server and the pings came back successfully so the route had to be in place. 

This is where I used AI to help me troubleshoot. I was told to check the firewall but `firewalld` wasn't even installed to use `firewall-cmd` commands. AI had me use the iptables to troubleshoot. The default policy was allow and we didn't see anything specifically stating to block the traffic. AI gave me a rule to implement and once I added the rule, I was able to curl from the working server to the broken server. Now, I'm not familiar with iptables so this would've stumped me. Nonetheless, it was a good learning experience!


## Day 11: Install and Configure Tomcat Server

I'll take to look back over this one. Installing the Tomcat server was pretty straightforward but they wanted me to configure it in a specific way. I had to google that part as I wasn't familiar with this type of server. 
