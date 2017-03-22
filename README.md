# EBS Snapshotter

Command line tool for creating Amazon EBS snapshots for volumes in one or more EC2 regions and automatically removing older snapshots.

## Installation

Download a binary from the [releases area](https://github.com/HealthcareBlocks/ebs_snapshotter/releases) or build locally (see below).

## Usage

Snapshot every volume in the same region as the host machine, keeping last 7 snapshots (default) for each volume. Region is inferred using EC2 metadata:
```
ebs_snapshotter
```

Snapshot every volume in the same region as the host machine, keeping last 5 snapshots for each volume:
```
ebs_snapshotter -retain=5
```

You can also specify regions explicitly if the host machine needs to snapshot volumes in multiple region(s):

```
ebs_snapshotter -regions=us-east-1,us-west-2 -retain=5
```

Generating an SNS alert for each region after completion:
```
ebs_snapshotter -regions=us-east-1,us-west-2 -sns_topic="arn:aws:sns:us-west-2:123456789:BackupAlerts"
```

Overriding SNS alert details with your own values:
```
ebs_snapshotter -regions=us-east-1 -sns_topic="arn:aws:sns:us-west-2:123456789:BackupAlerts" -sns_subject="Snapshots Process" -sns_message="us-east-1 complete"
```

Run ```ebs_snapshotter -h``` to view all options.

### Volume Tags are Automatically Copied to Snapshots

To disable this behavior, set ```-copytags=false```.

### Default Region

If ```-regions``` or ```-snsRegion``` are omitted, this tool will use the host machine's EC2 metadata to populate these values. Thus, if running this tool from a non-EC2 machine, be sure you set these values.


## Production Usage

This tool can be installed directly on an EC2 instance and scheduled via cron. An alternate approach is to use AWS Lambda, see [this post](http://docs.aws.amazon.com/lambda/latest/dg/with-scheduled-events.html).

### Example Crontab

Backup at 1 AM nightly:
```
0 1 * * * /bin/ebs_snapshotter -regions=us-east-1 -retain=5 -sns_topic="arn:aws:sns:us-west-2:123456789:BackupAlerts" > /var/log/cron.log
```

### Docker

A sample [Dockerfile](Dockerfile) is included in this repo for reference. Or use the existing image:

```
docker pull healthcareblocks/ebs_snapshotter
```

The binary is used as the entrypoint, so you can pass options directly:

Example:
```
docker run --rm healthcareblocks/ebs_snapshotter -retain=5
```

## AWS Authentication

The host machine should be configured for AWS access either via an IAM role or
a user with AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY set in the environment.

## AWS IAM Permissions

* ec2:CopySnapshot
* ec2:CreateSnapshot
* ec2:CreateTags
* ec2:DeleteSnapshot
* ec2:DeleteTags
* ec2:DescribeSnapshotAttribute
* ec2:DescribeSnapshots
* ec2:DescribeTags
* ec2:DescribeVolumes
* ec2:ModifySnapshotAttribute
* ec2:ResetSnapshotAttribute
* SNS:Publish [optional - applicable if sending SNS messages]

## Building Locally

Builds are generated by a Docker image and output to a bin directory, see [makefile](Makefile).

Dependencies are managed via https://github.com/golang/dep
