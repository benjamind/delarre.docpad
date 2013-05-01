---
title: Elastic Beanstalk and VPC Fun
layout: post
tags: ['aws','elastic beanstalk','elb','vpc','configuration']
date: 05/01/2013
---
So last time I blogged about getting libcouchbase installed on an Elastic Beanstalk deployment. That was a major headscratching session there. After we got that working I had to move on to getting our EB configuration to deploy within a Virtual Private Cloud. This was a bit of a nightmare...

### Looks easy at first
First out it looks like this should be relatively easy. We just setup a configuration file in our `.ebextensions` folder that can configure the VPC settings using our first configuration file which I'll name `01_environment.config`:

```
option_settings:
  - namespace: aws:autoscaling:launchconfiguration
    option_name: EC2KeyName
    value: ec2keypair
  # Now setup the VPC that our EC2 instances should use
  - namespace: aws:ec2:vpc
    option_name: VPCId
    value: vpc-un1que1d
  # Now we setup a subnet for our VPC ec2 instances to use
  - namespace: aws:ec2:vpc
    option_name: Subnets
    value: subnet-un1que1d
  # And another subnet for our ELB
  - namespace: aws:ec2:vpc
    option_name: ELBSubnets
    value: subnet-un1que1d2
  # Now set the instance type we want to use for autoscaling
  - namespace: aws:autoscaling:launchconfiguration
    option_name: InstanceType
    value: m1.small
  # And setup a security group for NAT
  - namespace: aws:autoscaling:launchconfiguration
    option_name: SecurityGroups
    value: sg-un1qu1d3
```

This is all mostly straight out of the documentation for EB which you can [find here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo-vpc-basic.html). Of course, its not as easy as all that. Once all this is in place and you do a new deployment with EB you can see that this doesn't actually work! It appears that the security group that is deployed on the Elastic Load Balancer isn't correctly configured to allow traffic from port 80 to port 8080 inside the VPC. This means that while all the servers come up inside the VPC correctly, there's no external access to them so our web services can't be accessed from the outside world. This is probably a sensible configuration for a VPC to have since its supposed to be Private after all, but its no good for our services.

### Lacking documentation
So I set out to find some documentation that would tell me how to configure the security group for the ELB that Elastic Beanstalk creates for me. Unfortunately it appears that you can't do this...there's nothing in the documentation for Elastic Beanstalk about configuring the ELB or very much about how to configure any of the resources that EB brings up for you. However...all is not as it seems!

ELB uses the same configuration management and resource setup as the CloudFormation service. So while its not actually documented, all configuration you can do in CloudFormation configuration files can pretty much be done with files in your .ebextensions folder. So lets setup a SecurityGroup using [this documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html) and configure our LoadBalancer using [this documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-elb.html). This gives us a file which I'll name `02_load_balancer.config` which looks a little like this:

```json
"Resources" : {
  "AWSEBLoadBalancerSecurityGroup": {
    "Type" : "AWS::EC2::SecurityGroup",
    "Properties" : {
      "GroupDescription" : "Enable 80 inbound and 8080 outbound",
      "VpcId": "vpc-un1que1d",
      "SecurityGroupIngress" : [ {
        "IpProtocol" : "tcp",
        "FromPort" : "80",
        "ToPort" : "80",
        "CidrIp" : "0.0.0.0/0"
      }],
      "SecurityGroupEgress": [ {
        "IpProtocol" : "tcp",
        "FromPort" : "8080",
        "ToPort" : "8080",
        "CidrIp" : "0.0.0.0/0"
      } ]
    }
  },
  "AWSEBLoadBalancer" : {
    "Type" : "AWS::ElasticLoadBalancing::LoadBalancer",
    "Properties" : {
      "Subnets": ["subnet-un1que1d2"],
      "Listeners" : [ {
        "LoadBalancerPort" : "80",
        "InstancePort" : "8080",
        "Protocol" : "HTTP"
      } ]
    }
  }
}
```

Just ensure that your `VpcId` matches the one in your earlier configuration, and that the Subnet configuration matches the one in ELBSubnets configuration.

Now when you deploy you should see it create this new security group for you and automatically assign the load balancer to it, thus allowing you to access your web services. Its great that Amazon provide all these services, but really there's a lot of hidden knowledge locked up in the documentation thats not clearly obvious anywhere. The ability to use all the CloudFormation configuration options in your Elastic Beanstalk `.ebextensions` files is frankly awesome opening up a huge range of possibilities for Elastic Beanstalk deployments.