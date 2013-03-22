---
title: Deploying libcouchbase with AWS Elastic Beanstalk
layout: post
tags: ['aws','elastic beanstalk','elb','libcouchbase','couchbase']
date: 03/22/2013
---
Recently we've begun using couchbase as a backend datastore for some of our projects at my day job with SupplyFrame. Since my projects are now involving nodejs I needed to make us of the [couchnode](https://github.com/couchbase/couchnode) package. Unfortunately this depends upon [libcouchbase](http://www.couchbase.com/develop/c/current) being installed on the platform before the package is installed.

We're using [Elastic Beanstalk](http://aws.amazon.com/elasticbeanstalk/) for our deployments so I needed to figure out how to get the `libcouchbase` libraries installed before `npm install` was run so it would install correctly. Thankfully Amazon have figured this would be a problem and so have provided a rather handy mechanism for configuring your AWS environments under Elastic Beanstalk.

### .ebextensions configurations
If you include a folder called `.ebextensions` in the root of your source package (in our case a git repo), then you can add configuration files (with the extension `.config`) there that will be executed in alphabetical order during environment start. Full documentation for the `.config` files can be [found here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html).

If you're using one of the standard packages and the packages you need are available in the standard repositories then you can probably make use of the [`packages`](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#customize-containers-format-packages) key to automatically install the packages you need. This would look something like this inside your `01_myapp.config` file (below example taken from the docs):

```
packages:
  yum:
    libmemcached: []
    ruby-devel: []
    gcc: []
  rpm:
    epel: http://download.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm
  rubygems:
    chef: '0.10.2'
  apt:
    mysql-client: []
```

Unfortunately `libcouchbase` is not in the standard repositories, and to make matters worse the Amazon Linux AMI we're using doesn't even have some of the dependencies for this package installed. So I had to get a little more creative.

### Using .ebextensions commands
There's another key called `commands` in the `.config` specification that allows you to execute arbitrary shell commands as part of the environment setup. This took a bit of fiddling to get going, but after much headscratching I finally came up with the following which seems to do the job and get `libcouchbase` and all its dependencies installed:

```
# Errors get logged to /var/log/cfn-init.log. See Also /var/log/eb-tools.log
commands:
    01-command:
        command: wget -O/etc/yum.repos.d/couchbase.repo http://packages.couchbase.com/rpm/couchbase-centos55-x86_64.repo

    02-command:
        command: yum install epel-release
        ignoreErrors: true

    03-command:
        command: yum check-update
        ignoreErrors: true

    04-command:
        command: yum install -y --enablerepo=epel libev

    05-command:
        command: yum install -y libcouchbase2-libevent libcouchbase2 libcouchbase-devel
```

What this does is mostly described in the `libcouchbase` installation instructions, with a couple of extra additions, `02-command` ensures that our AMI has the `epel` repository installed in yum so that we can gain access to the `libev` package installed in `04-command`. Finally we modified the main install command to include the `libcouchbase2-libevent` package which appears to have been missed from the dependency hierarchy for `libcouchbase2`.

### Gotchas
This should have been really straightforward to get going, unfortunately as always, there were a few hiccups that made this all take much longer to figure out. Firstly these `.config` files are supposed to be YAML format, but they appear to be very very sensitive to syntax issues and to make matters worse if you get the syntax slightly wrong you'll probably see your Elastic Beanstalk environment just lock up for about 40 mins before reverting to the last working version. Secondly it can be pretty hard to debug whats gone wrong, especially if the instance reverts to the last good version since quite often the process of reverting will wipe out most of the log messages you need to see to figure out whats wrong. Also you don't always get the best log messages so it can be pretty tough to figure out whats going on.

However, I have to say I'm quite happy with the setup now, I can easily push my local git repository into Elastic Beanstalk and have all my dependencies setup in one easy command. Great stuff. Next time I'll detail some of the perils of trying to do all this inside a VPC!