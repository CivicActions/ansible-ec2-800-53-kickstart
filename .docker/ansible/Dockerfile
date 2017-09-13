FROM centos:centos7

# Install required development packages.
RUN yum -y update && yum -y install epel-release && yum clean all
RUN yum -y install gcc make python-devel libffi-devel openssl-devel openssh-clients git && yum clean all
RUN curl -O https://bootstrap.pypa.io/get-pip.py
RUN python get-pip.py

# Install ansible with pip.
RUN pip install ansible ansible-lint awscli boto boto3

RUN mkdir -p /opt/ansible
WORKDIR /opt/ansible

# Show version.
RUN ansible --version
