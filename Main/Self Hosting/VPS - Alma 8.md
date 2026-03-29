
# **Operations as root**

passwd #change root password

adduser jubin

passwd jubin

usermod -aG wheel jubin #Add jubin to sudo group

# **Login as jubin**

hostnamectl set-hostname <hostname> # to set the hostname.

yum update -y

## Needed for building node.js projects which use libraries such as sqlite that has python dependency

sudo dnf install -y python3 python3-devel make gcc gcc-c++ redhat-rpm-config

### Enable newer GCC via DevToolset

sudo yum install -y centos-release-scl sudo yum install -y devtoolset-11

# Firewall Fixes

# Masquerading allows for docker ingress and egress (this is the juicy bit)

firewall-cmd --zone=public --add-masquerade --permanent

# Specifically allow incoming traffic on port 80/443 (nothing new here)

firewall-cmd --zone=public --add-port=80/tcp

firewall-cmd --zone=public --add-port=443/tcp

# Reload firewall

firewall-cmd --reload

#Make changes permanent

firewall-cmd --runtime-to-permanent # or add –permanent to the firewall add port command

[Reverse Proxy](https://www.notion.so/Reverse-Proxy-2d45278ac26780f8a352e7225d22cb26?pvs=21)

[Reverse Proxy](https://www.notion.so/Reverse-Proxy-2ea5278ac26780699b08eaac777b1f3d?pvs=21)