# awsible
Create &amp; Manage EC2 instances as "projects" with Ansible

# The Problem
I have two use-cases for AWS:

1. verifying claims made about OS configurations, currently solved quickly by having long-running tiny instances of each common OS. These fall behind in patches and need to be remade to be based on the latest AMI as the Quick Start list of AMIs get updated in the EC2 Launch Instance Wizard. These instances have static entries in my local ~/.ssh/config that also drift out of date, and cost quite a bit to keep running. I've been calling these "bastions".
2. building and tearing down separate "stacks" to reproduce customer issues with VPC & EC2

Both are manual, and neither fully utilize the free & cheap services that greatly improve the EC2 experience:

1. Project tagging with Resource Groups in the AWS Console.
2. CloudWatch Alarms w/ the free email and the ~100 SMS messages for very anomalous behavior
3. CloudWatch Alarm on System Status Check for the Recovery action - no more manual action on Maintenance Notifications!
4. "shipping" low-frequency OS logs to CloudWatch Logs or an ELK stack
5. Memory usage sent to CloudWatch Metrics as a custom metric (using the publicly available Perl scripts)
6. separate SSH keys per project

# What I want

1. `awsible rebuild bastion amazonlinux 2017.03` should terminate my last Amazon Linux "bastion" and replay any changes off of default that I recorded, on top of a new instance using the "2017.03" version AMI in my default region
2. `awsible do once -shell "uname -a" on amazonlinux` finds my bastion of Amazon Linux in its list, and runs `ansible -m shell "uname -a"` on it and returns only the result.
3. `awsible project init` make the current directory into a templated Ansible repo
4. `awsible project up` run the ansible repository, basically `ansible-playbook site.yml` in the directory, feeding in the right AWS credentials & inventory
5. `awsible do always -ansible "-m package -a name=yum-cron" on amazonlinux,rhel6,rhel7,centos6,centos7` should attempt to run the Ansible ad-hoc command on the bastions of each OS, if they succeed, add the command to a playbook per OS to be run, run the playbook expecting no changes, then terminate the bastions and rebuild them using the playbook.

# What should it look like?
As much Ansible and as little raw Python as possible, but do it right: unit tests that run in Travis-CI, tag and cut GitHub releases, ReadTheDocs website (GitHub Wiki is meh and versions don't tie to commits/branches)

Q. Why not CloudFormation?
A. I would still want a templating system to build them for me, and run them across regions, but Ansible can do the "run on everything"/pssh/sshuttle orchestration and CloudFormation can't

# References

[ec2-vpc-example.yml]( https://github.com/ansiblebook/ansiblebook/blob/1st-edition/ch12/playbooks/ec2-vpc-example.yml )


NOTE: ansible inventory has drastically changed, instead use a yaml file to provide config for a new built-in aws_ec2 plugin, and see the results with `ansible-inventory -i demo.aws_ec2.yml --graph`
