---
title: "Quick and Easy MapReduce Jobs Using Amazon AWS and Node.JS with EMR"
posted: 2015-10-19T16:21
tags: [emr, mapReduce, hadoop]
track: Technical
thumbnail: aws-hadoop-node.jpg
layout: article-layout.html
---
As part of a data analytics course with Udacity I recently took, I had the chance to play a bit with Hadoop. It seems like I never want to take the easy path and just do the assignment, as this seemed liked the perfect chance to *also* experiment with Amazon’s Elastic MapReduce (EMR). And of course, why not *also* experiment with using node.js as well. It turns out after a bit of tweaking command line paramaters, it is actually relatively easy to spin up a EMR cluster, send it some jobs, and then shut it down.

## Command Line Setup
To follow my examples in this post you will need to have [Amazon AWS CLI](https://aws.amazon.com/cli/) installed and working. If you are mac or linux, you can install with a pip.

`pip install awscli`

After installation follow the directions to setup the environment variables for your credentials. If you have multiple amazon accounts, perhaps for different projects or work and personal accounts, I highly recommend a quick look at [Named Profiles](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-multiple-profiles) as an easy way to switch back and forth between profiles.

## Send Data to S3
Using Amazon’s Elastic MapReduce is surprisingly simple when it comes to data storage. There is no need to think about a custom file system or how data might be split across nodes.

All data is just uploaded to Amazon S3 storage, and each node has access to all of the data. This is one aspect where Amazon has greatly simplified the MapReduce experience.

To copy data using the cli, just use a command like: `aws s3 cp purchases.txt s3://rurri-hadoop/purchases.txt`.

## Start a cluster
The first step in running some MapReduce jobs is to start a cluster. Another benefit of using Amazon’s Elastic MapReduce is that we can turn on quite a large cluster for some jobs, and then turn it off when it is done and not incur a great deal of costs.

### Spot Instances
This is a good time to check out [spot instance pricing](https://aws.amazon.com/ec2/spot/pricing/). MapReduce jobs lend themselves especially well to Spot Instances. You can save a great deal of money just by using this option with your jobs, and there is usually very little downside to using Spot instances for things like a MapReduce job. The examples below all use spot instance pricing.

### Start the cluster with the CLI
We do this at the command line.

```
aws emr create-cluster --enable-debugging --visible-to-all-users --name MyNodeJsMapReduceCluster --instance-groups  InstanceCount=3,InstanceGroupType=CORE,InstanceType=m3.xlarge,BidPrice=0.25 InstanceCount=1,InstanceGroupType=MASTER,InstanceType=m3.xlarge,BidPrice=0.25 --no-auto-terminate --enable-debugging --log-uri s3://rurri-hadoop-logs --bootstrap-actions Path=s3://github-emr-bootstrap-actions/node/install-nodejs.sh,Name=InstallNode.js --ec2-attributes KeyName=iSSH --ami-version 3.9.0
```

Be sure to modify the command line and replace the following values:
- log location bucket
- instance size (count) for both master and nodes
- bid price for both master and nodes
- a KeyName matching a keyname under amazon’s EC2 if you want to be able to log into the instances directly and troubleshoot.

This command will start up an EMR cluster. You can verify that it exists by logging into the AWS Console.

This method will return the id of a cluster. It should look something like this: `j-V570752TSQXF`. Note this id, as we will need this cluster id for sending jobs and for terminating the cluster.

**Important:** You will begin accruing charges up to the bid price per node once the create cluster command is issued. Don’t forget to turn the cluster off when you are done, unless you want to continue paying for the convenience of not having to wait for a new cluster to spin up for future jobs. Assuming this is for personal experimentation--you will want to shut it off.

## Creating a Job
To create a job, we need to first make the mapper and reducer node.js files available via s3. Easiest way for me to do this is to just use the aws cli. To copy these files in my example I used the command: `aws s3 cp mapper.js s3://rurri-hadoop/mapper.js` followed by `was s3 cp reducer.js s3://rurri-hadoop/reducer.js`.

Once our files are up and ready, we can issue the command for the job.

```
was ear add-steps —cluster-id j-V570752TSQXF —steps Name=NodeJSStreamProcess,Type=Streaming,Args=—files,”s3://rurri-hadoop/mapper.js\,s3://rurri-hadoop/reducer.js”,-input,s3://rurri-hadoop/purchases.txt,-output,s3://rurri-hadoop/step1,-mapper,mapper.js,-reducer,reducer.js
```

Be sure to change the values for:
- Cluster ID
- bucket name and file name for mapper
- bucket name and file name for reducer
- bucket name and file name of input data
- bucket name and location for output data.

## Accessing Results
The job we just created can be viewed in the Amazon Console under our EMR cluster we created. Better yet is to just pull the results so that we can do things with them. In this example we stored the results in the s3 bucket with the prefix step1. We can grab the contents of that directory with the cli command: `s3://rurri-hadoop/step1 .`

We can now view the results all at once with: `cat part-* | sort`

## Terminating the Cluster
Don’t forget to shut things down. We just need the cluster id from earlier:
```aws emr terminate-clusters —cluster-ids j-V570752TSQXF```

We probably don’t want to just take the CLI’s word for it initially. Would be worth to double check in the Web Console that we don’t have any clusters running and accumulating a hefty bill.

## Testing our Map/Reduce Jobs
A great trick for testing out map reduce jobs prior to running the whole thing was taught in the Udacity course.

Test the mapper:
`head purchases.txt -n 100 | nodejs mapper.js`

Test the reducer:
`head purchases.txt -n 100 | sort | nodejs reducer.js`

## Troubleshooting
When first starting, I found it best to keep a window with the web AWS console open so that I could see the status of jobs and to be able to view their associated logs quickly.

Jobs can sometimes take longer than you think to fail, even if it is a failure in the bootstrap of the job, it can sometimes take over a minute just to get the failure reported. This means you can’t rule out the bootstrap process as the reason for the failure, even if it took some time for it to be reported.


