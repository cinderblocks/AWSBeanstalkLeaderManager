# AWSBeanstalkLeaderManager

A configuration file for your amazon elastic beanstalk that makes sure that one instance in your pool always is tagged as "leader". The ruby part of this configuration is inspired by [whenever-elasticbeanstalk](https://github.com/dignoe/whenever-elasticbeanstalk) (what was too komplex for our purpose).

## Installation

Just download the source code and copy the `leader-manager.config` into your `.ebextensions` folder. The final path may be `your-app-root/.ebextensions/leader-manager.config`. `leader-manager.rb` and `leader-manager-crontab.txt` are only included for a better overview.

Everything should work out of the box.

## EC2 Instance IAM Role Permissions

As described in [whenever-elasticbeanstalk/README.md](https://github.com/dignoe/whenever-elasticbeanstalk/blob/master/README.md) you have to add at least the following policy once:

	{
	  "Version": "2015-03-29",
	  "Statement": [
	    {
	      "Action": [
	        "ec2:DescribeInstanceAttribute",
	        "ec2:DescribeInstanceStatus",
	        "ec2:DescribeInstances",
	        "ec2:DescribeTags",
	        "ec2:CreateTags"
	      ],
	      "Resource": [
	        "*"
	      ],
	      "Effect": "Allow"
	    }
	  ]
	}

## Usage and benefit

This solution has two benifits for you:

1.  **Tagged instance**

    At least one instance will be tagged as `leader` by using the tag `leader=true`. There may be also instances using the tag `leader=false` since it is theoretically possible that two instances are leader for a small amout of time (1 minute). 

	![img_252_-_735x202 1](https://cloud.githubusercontent.com/assets/5159398/6886991/dcda4288-d652-11e4-84e5-5eb3c3205f23.png)


2.  **Local detection**

    Since it would be a big overhead to check for the leader flag in your application, this script will touch a file called `leader-manager-is-leader` in `/var/app`. So its very easy for you to detect whether you are running code on the leader instance.

# Examples

We wrote this solution since it is a "known problem" that an Elastic Beanstalk will terminate the leader instance since the "leader" flag is only available while deploying.

The main apporach of this script is to make sure that cronjobs only run on one instance. We do this using this bashscript:

	if test -e "/var/app/leader/leader-manager-is-leader"; then
	    # do something cool
		# e.g.: php /var/app/current/app/console some:symfony2:console:command
	fi

You can read more about this problem [here](http://stackoverflow.com/questions/14077095/aws-elastic-beanstalk-running-a-cronjob), [here](http://blog.paulopoiati.com/2013/08/25/running-cron-in-elastic-beanstalk-auto-scaling-environment/) and [here](https://forums.aws.amazon.com/thread.jspa?threadID=130481).