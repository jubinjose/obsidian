
#### ==**These are steps needed when the node project uses a library that needs c++/python for build such as sqlite**==

# CentOS/RHEL 8 / Stream / Rocky / Alma
sudo dnf install -y python3 python3-devel make gcc gcc-c++ redhat-rpm-config


Enable newer GCC via DevToolset
sudo yum install -y centos-release-scl
sudo yum install -y devtoolset-11

scl enable gcc-toolset-11 bash
export PYTHON=/usr/bin/python3
npm ci