# Installing and using Polorproxy on AWS EC2 Instance running in a Docker container
## Overview
This guide will show how to set up Polorproxy on a AWS EC2 Instance running in a Docker container for TLS traffic inspection.
* This guide does not go over setting up an AWS EC2 Instance
* This guide does not go over setting up docker
* I am using Amazon Linux Distro 

Refrences: 
https://www.netresec.com/?page=Blog&month=2020-10&post=PolarProxy-in-Docker

![AWS Diagram](https://github.com/patclarke/TLS-traffic-decryption/blob/main/Images/AWS_Diagram.png)
![Proxy Diagram](https://github.com/patclarke/TLS-traffic-decryption/blob/main/Images/Proxy_Diagram.png)


## Stection 1 Docker Container
Create the dockerfile with a text editor
```
vim Dockerfile
```
Paste the following into the empty dockerfile and save it.
```
FROM mcr.microsoft.com/dotnet/core/runtime:2.2
EXPOSE 10443
EXPOSE 10080
EXPOSE 57012
RUN groupadd -g 31337 polarproxy && useradd -m -u 31337 -g polarproxy polarproxy && mkdir -p /var/log/PolarProxy /opt/polarproxy && chown polarproxy:polarproxy /var/log/PolarProxy && curl -s https://www.netresec.com/?download=PolarProxy | tar -xzf - -C /opt/polarproxy
VOLUME ["/var/log/PolarProxy/", "/home/polarproxy/"]
USER polarproxy
WORKDIR /opt/polarproxy/
ENTRYPOINT ["dotnet", "PolarProxy.dll"]
CMD ["-v", "-p", "10443,80,443", "-o", "/var/log/PolarProxy/", "--certhttp", "10080", "--pcapoverip", "0.0.0.0:57012"]
```
Now build the docker image 
```
polarproxy-image .
```
Then we are going to create the docker container 
```
docker create -p 443:10443 -p 10443:10443 -p 10080:10080 --name polarproxy polarproxy-image
```
And now start the container 
```
docker start polarproxy
```
Confirm polarproxy is running 
```
docker ps
docker logs polarproxy
```

## Section 2 AWS Security Group Inbound Rules
In the AWS console navigate to the Security group used when setting up your EC2 instance 

EC2 > Security Group > Group Name

We are going to "Edit inbound rules" and add the following. Replace $PublicIP with the public IP of the device that will be forwarding its traffic to the proxy.
| Type       |      Protocol      |  Port range | Source    |
|------------|:------------------:|------------:|----------:|
| HTTPS      |  TCP               | 443         | $PublicIP |
| Custom TCP |  TCP               | 10080       | $PublicIP |
| Custom TCP | TCP                | 10443       | $PublicIP |

Confirm the client can reach the EC2 instance on port 443. Replace $EC2IP with the IP or domain name of the EC2 instance. 
```
nc -zv $EC2IP 443
```
You should see somthing like this
```
Connection to ec2-1-1-2-2.us-northeast-3.compute.amazonaws.com 443 port [tcp/https] succeeded!
```

## Section 3 Root CA 
## Section 4 SOCKS proxy
## Section 5 TLS inspection
