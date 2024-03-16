# PROJECT 8 - LOAD BALANCER SOLUTION WITH APACHE

![project8](./images/1.png)

![project8](./images/3.png)

![project8](./images/4.png)

![project8](./images/5.png)

![project8](./images/6.png)

## CONFIGURE APACHE AS A LOAD BALANCER

- Create an Ubuntu Server 20.04 EC2 instance and name it `Project-8-apache-lb`, so your EC2 list will look like this:

![project8](./images/7.png)

![project8](./images/8.png)

- Open TCP port 80 on `Project-8-apache-lb` by creating an Inbound Rule in Security Group.
- Install Apache Load Balancer on `Project-8-apache-lb` server and configure it to point traffic coming to LB to both Web Servers:

```bash
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```

- Make sure apache2 is up and running `sudo systemctl status apache2`

- Configure load balancing by editing `sudo vi /etc/apache2/sites-available/000-default.conf` file

```config
#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
    BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
    BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
    ProxySet lbmethod=bytraffic
    # ProxySet lbmethod=byrequests
</Proxy>

ProxyPreserveHost On
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```

![project8](./images/9.png)

- Restart apache server `sudo systemctl restart apache2`

![project8](./images/10.png)

> Bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

> Other methods, include: bybusyness, byrequests, heartbeat

- Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
  `http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

![project8](./images/11.png)

> Note: If in the Project 7 you mounted `/var/log/httpd/` from your Web Servers to the `NFS server` – unmount them and make sure that each Web Server has its own log directory.

- Open two ssh/Putty consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

- Try to refresh your browser page `http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php` several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set `loadfactor` to the same value for both servers – it means that traffic will be disctributed evenly between them.

![project8](./images/12.png)

![project8](./images/13.png)

> If you have configured everything correctly – your users will not even notice that their requests are served by more than one server.

## OPTIONAL STEP – CONFIGURE LOCAL DNS NAMES RESOLUTION

> Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
> What we can do, is to configure local domain name resolution. The easiest way is to use `/etc/hosts file`, although this approach is not very > scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.

- Open this file on your LB server

`sudo vi /etc/hosts`

- Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

```config
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

- Now you can update your LB config file with those names instead of IP addresses.

```config
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

- You can try to `curl` your Web Servers from LB locally `curl http://Web1` or `curl http://Web2` – it will work.

> Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

- #### THIS VIDEO CONTAINS ALL COMMANDS USED TO FINISH THIS PROJECT - [Project 8 Video](https://drive.google.com/file/d/1sNyUN7UwtxJ9nCjMQrLJfjAjxSZTKdKZ/view?usp=sharing)

- Now your set up looks like this:

![project8](./images/14.png)

Congratulations!
You have just implemented a Load balancing Web Solution for your DevOps team.
