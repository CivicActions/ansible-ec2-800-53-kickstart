# Ansible

## Setup

* You will need accounts and working access on all servers to start.
* You will need Docker installed and working. Ansible is not required (it only runs inside Docker).
* Create an env.local file (as follows) with:
 * Your SSH username (replace USERNAME below).
 * The name of an SSH key configured in your AWS account as described in https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html (replace KEYNAME below).

```
ANSIBLE_USER=USERNAME
AWS_SSH_KEY_ID=KEYNAME
```

* You will need AWS access. Login to your AWS account and create API access keys - these should be saved into `$HOME/.aws/credentials` under an new group named after the project (replace SECRET_KEY and ACCESS_KEY below):

```
[project]
region=us-east-1
aws_secret_access_key = SECRET_KEY
aws_access_key_id = ACCESS_KEY
```

### Initial setup

The following only needs to be done once upon creation of the project:
* Set up an IAM group for admin users, create accounts and share account credentials.
* Update the env.list file with the following:
 * The project name (in lowercase) to match the group name that will be referenced in users  `$HOME/.aws/credentials` files (replace PROJECT below).
 * The AWS region to use (replace REGION below).

```
# These environment variables are made available to Ansible and AWS CLI
AWS_PROFILE=PROJECT
AWS_REGION=REGION
EC2_REGION=REGION
```

## Usage

* Run `. bin/activate` to start using Ansible - this will do the following:
    * Build the Docker image.
    * Validate some basic configuration.
    * Mount the configuration, local configuration files and your SSH agent socket into the container.
    * Add environment variables from env.list and env.local to the container environment.
    * Create aliases for various Ansible commands with some basic default options - run `cat bin/activate` to view source.

## Inventory

* To view manually managed groups of hosts, review files in the inventory directory.
* To view manually managed host variables, review yaml files in the inventory/host_vars directory.
* To view manually managed group variables, review yaml files in the inventory/group_vars directory.
* The inventory/group_vars/all.yml file contains user directories and IP whitelists.
* To view a dynamic list of running hosts from AWS and their ec2 metadata, run `inventory`

## Playbook structure

* `site.yml` this is the primary playbook and is where any configuration that applies to all hosts is managed. This deals with instance provisioning, managing instance specific EC2 configuration as well as managing hardening and users on ansible managed hosts. Tags are used to allow skipping of slow tasks if needed.
* `vpc.yml` manages all overall VPC configuration, setting up production and development VPCs and managing subnets and available security groups for each.
* All other `*server.yml` top level playbooks manage configuration for specific server roles (e.g. webserver, mailserver etc). This includes both role specific EC2 configuration as well as server configuration for Ansible managed servers (or aspects of configuration that has been taken over from Puppet).
* The `utility_plays` directory contains utility playbooks - these should not themselves specify any permanent effect on an instance (i.e. they should not be used to do any configuration themselves, but can gather information or affect temporary changes like reboots).

Avoid miscellanious playbooks to manage configuration, since these make it hard to understand the overall state on a particular server at a point in time. Instead all configuration should be managed via the site.yml playbook or the appropriate `*server.yml` server role playbook.

## Common tasks

* To check connectivity to each server run `ansible GROUP -m ping` where GROUP is a group or specific server(s).