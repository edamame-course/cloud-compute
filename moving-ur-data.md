# How do I move my data to the cloud?

Authored by Adina Howe for EDAMAME2016

## Summary

### Overarching Goal
* This tutorial will teach you how to move your data into a more permanent place on EC2

### Learning Objectives
* Understanding how to create a volume
* Understanding how to format a volume
* Understanding how to attach a volume
* Understanding how to move data onto an attached volume

### Tutorial

Ok, let's start with defining some terms.  You already know what an EC2 instance is - its your cloud computer.  We are now going to be working with an EC2 EBS (Elastic Block Storage) volume.  Think of this as a giant external hard drive or USB flash drive storage system.  It is external to your cloud computer and needs to be prepped before storing data on it.  The steps are as follows:

1.  Purchase / create an EBS volume
2.  Attach the volume to an instance
3.  Format the volume 
4.  Mount the volume to a directory of your choice, e.g., make it accessible

## Start the cloud computer - any one will do.
1.  Start an EC2 - feel like a pro

Important:  Note where the location/availability of this instance is on your dashboard (e.g., US-east-1a).  Click on the lil box next to your instance and look at the details.

## Creating an EBS Volume
The official instructions for this are here at this [link](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html).  Its a good guide if any of the following confuses you.  

1.  Navigate on a web browser to your Amazon AWS Dashboard.  
2.  From the navigation bar in the upper right corner, make sure you are in the region you created your instance (e.g., Northern Virginia).
3.  In the navigation pane on the left, under ELASTIC BLOCK STORE, choose Volumes.  
4.  In the next screen, above the upper pane, choose Create Volume.
5.  For this class, keep all defaults but do change the SIZE to fit your data and analysis needs.  My rec:  200G minimum, 500G metagenomes.
6.  Select the right availability zone (the same as the instance that you noted above).
7.  Choose "Create".

You've just purchased the equivalent of a giant flash drive that we'll now poke into your cloud computer.

## Poking - or attaching your volume

Here is the [link](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html) to this official documentation.

1.  Select a volume and choose Attach Volume.
2.  In the Attach Volume dialog box, start typing the name or ID of the instance to attach the volume to for Instance, and select it from the list of suggestion options (only instances that are in the same Availability Zone as the volume are displayed).  Note:  if you click in the box, Amazon will try to help you fill it in.
3.  You'll note that Amazon gives you a suggestion on the device -- keep it and note it.  e.g., /dev/sdf.

## Connect to your instance

Remember:  ssh -i <yourkeyfile> ubuntu@publicdns.amazon.com

Example:  

ssh -i adina.pem ubuntu@ec2-54-90-239-39.compute-1.amazonaws.com

## Check to see if your volume is attached

Here is the official [documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)

In the command line, type:

```
lsblk
```

This tells you what volumes are available - you'll see one with lots of Gigabytes!!!  That's yours.  Look to the left of the e.g., 100G (or whatever size yours is) and you'll see a NAME like xvdf.  This is related to the device that Amazon recommended above, e.g., /dev/sdf.  Its changed now to e.g., /dev/xvdf.  Read the documentation if you want to really know why - its not that important for this tutorial.

Ok, it exists.  If not, get your pink panic stickie out and look really sweet and approachable for a TA.

## Format your volume

First, note here that I am assuming that your volume is on the device /dev/sdf now /dev/xvdf.  It could also be /dev/sdg and now /dev/xvdg or /dev/sdh and now /dev/xvdh (see the incrementation?).  Anyways, note it and make the appropriate changes below if you are not /dev/xvdf.

In the command line, type the following to format your disk.  Again, there's detials here  you can look up - like what file system am I formatting for?  You can read about these in the detailed manual but for this class, trust me, this is what you want.  

```
sudo mkfs -t ext4 /dev/xvdf
```

Breaking it down, sudo tells the computer your all-powerful (as if you typed in your owner password).  mkfs = format my drive.  -t ext4 = trust me this is a good format.  /dev/xvdf - this is the device name.

Now, type df.  What does this tell you do you think?

## And now...we'll make this volume accessible.

Make a directory for your volume.  Here's an example:

```
mkdir /data
```

Mount your volume:

```
mount /dev/xvdf /data
```

Type "df" again - and check it out.

Ok, now navigate /data.  How?

```
cd /data
```

The "/" is important note.  Now, release the permissions so that you can move files in and out (and anyone with the keyfile can).

```
chmod 777 /data
```

chmod = change permissions; 777 = everyone can do things; 

## Transferring data onto your volume.

Get back to your laptop -- open up a terminal or the Windows equivalent (the thing you use to sign onto EC2 instances with ssh).  We're going to use the scp command.

Remember:  

scp -i <security file> <from location> <to location>

Examples:  

scp -i adina.pem ubuntu@ec2-54-90-239-39.compute-1.amazonaws.com:~ubuntu/tutorials/data.txt ~/Downloads   (This is from my EC2 instance to my laptop)

scp -i adina.pem ~/Downloads/mycareer.docx ubuntu@ec2-54-90-239-39.compute-1.amazonaws.com:/data  (This is from my laptop to EC2)

Both these examples are executed from my laptop!

Ok, so find your data and scp-it to your instance.  It would be prudent to use wildcards, for example:

scp -i adina.pem ~/dataforedamame/*fastq ubuntu@ec2-54-90-239-39.compute-1.amazonaws.com:/data  (This is from my laptop to EC2)

This will transfer all fastq files in my dataforedamame directory.

If you are moving data from a website or ftp site, get its web link by right clicking and copying link and do something like the following (except with your link).  wget is a program to get data from a web url:

```
cd /data
wget https://vincentarelbundock.github.io/Rdatasets/csv/datasets/AirPassengers.csv
```

So....now you have your EBS volume.  This data in /data will not go away after you terminate yoru instance.  Note that data on your other non-EBS directories will terminate and disappear...yes...forever.  If you want to keep outputs from analysis, you'd want to keep it on /data or better yet, do all your analysis within the /data directory.  Also, note that you do have to pay for the hard drives on EBS - see [here](https://aws.amazon.com/ebs/details/).  Go forth!  Save the world!

