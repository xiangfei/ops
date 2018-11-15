euca-bundle-instance -b tingwu -p  windows.7.64.ta  -o ARN8TAXIFENJGCICY1IZP  -w 1mJUVPEk5X7zr6HJ2Nmb2y914KHXpOIy332z5iIf i-1EB43E03 

 euca-describe-bundle-tasks 


euca-register -d 'Description of Image' <bundle>/<prefix>.manifest.xml 
<bundle>  is tingwu 
<prefix> is windows.7.64.ta


Which option to choose?
Bundling a running instance
Bundling the image
Uploading the new image
Registering the image
 
Short cut by editing the rootFS in place – only for easy changes
There are two options for customizing an existing image. You can either bundle an existing instance into a new image; or you can customize the image rootFS file.

 

Which option to choose?
If you need to make large changes, or if you don’t have an existing rootFS, customize the image by bundling a running instance as described below. If you only need to copy data files or edit configuration files, you can also take the short cut.

 

Bundling a running instance
The easiest way to bundle a new image from a running instance is to use euca-bundle-vol on an existing server instance. See Eucalyptus documentation:

http://open.eucalyptus.com/participate/wiki/creating-image-existing-vm-centos
http://open.eucalyptus.com/participate/wiki/bundling-images
 

Prerequisites:

an instance that you have configurated
enough disk space (10 – 20 GB for Linux images)
kernel and ramdisk id’s (see below for more details where to get these)
a unique name for your new image
Eucalyptus credential zip on the instance
Then log into your instance, extract your credentials zip and source Euca rc:

# source .euca/eucarc
A good practice is to mount an EBS volume 3x size of the instances rootfs to /ebs for example. In the newer clouds you can also use /ephemeral.

 

In addition, you need to export the http proxy (remember to restore the http_proxy afterwards!):

# echo $http_proxy
http://87.254.212.120:8080/
 
# export http_proxy=
 

Bundling the image
To get the kernel and ramdisk IDs, run euca-describe-instances. Find your current instance and look for columns starting with eki- and eri-. These are the kernel and ramdisk IDs of the current instance, and you should use them.

Next, figure out a name. There is no recommended naming convention for private images. Use whatever suits your project or application. The image name has two components: the bucket name and the image prefix. Bucket names must be unique within the cloud, and prefix names must be unique within the same bucket. You can put several images in one bucket, if you want.

Then you bundle the image. Below is an example. In it, we create an image that has the prefix name RHEL53b32V7 (option –p) and store the bundle in the directory /ebs (option –d). The root file system has two gigabytes reserved for it (option –s).

There are some directories that should not be on the new image. These are specified with the –exclude option. In addition to the list below, you might add /ephemeral,/root/.ssh and any other directory that contains temporary or private files. Also, the –no-inherit option prevents metadata from the current instance ending up in the created bundle.

# euca-bundle-vol -p RHEL53b32V7 -d /ebs -s 2048 --exclude /ebs/,/var/run/dhclient-eth0.pid,/root/.euca --no-inherit --kernel eki-0DFA1532 --ramdisk eri-65D11676
Specify size in multiples of 512.

 

Uploading the new image
Once you have bundled the new image, you need to upload and register it to the cloud:

First upload the bundle into a Walrus bucket. You will need the bucket name (option –b) and a path to the manifest file created in the previous step (option –m). The manifest file is in the root of the directory you specified with the –d option. If you didn’t specify a directory, it is under /tmp/

You do not need to create the Walrus bucket beforehand. If the bucket does not exist, euca-upload-bundle will create it for you.

 

Input this command:
# euca-upload-bundle –b ProjectX -m /tmp/ RHEL53b32V7.img.manifest.xml
 
and you will get this in reply:
Checking bucket: ProjectX Creating bucket: ProjectX Uploading manifest file Uploading part: RHEL53b32V7.img.part.0 ----Clip----- Uploading part: RHEL53b32V7.img.part.16 Uploaded image as ProjectX/RHEL53b32V7.img.manifest.xml
 
You will need the name of the image from the last line of output to register your image.

 

Registering the image
Input this command:
# euca-register -d 'Description of Image' ProjectX/RHEL53b32V7.img.manifest.xml and you will get this in reply: IMAGE emi-F3F914C0
 

Don't forget to restore the http_proxy at the end:

# export http_proxy= http://87.254.212.120:8080/
 
Short cut by editing the rootFS in place – only for easy changes
Making changes to existing image root filesystem file is fairly simple. You can convert the bundle you created in the previous step into the rootFS image with the euca-unbundle command. This rootFS image is not encrypted and you can edit it in-place with normal Linux tools.

Once you have a rootFS image, loop mount it to some directory, go there and make the needed modifications to the filesystem:

# mount fedoraV11.10.x86-64.img new/ -o loop
# cd new/
# <Do your edits.>
 
After done, unmount the file:
# umount new
 

Then you can create a new bundle. Use the euca-bundle-image command in this case:?

# source eucarc
# export EKI=<Euca Kernel Image ID of the image, which is the same as in the original>
# export ERI=<Euca Ramdisk Image ID of the image, which is the same as in the original>
#
 

The Eucalyptus bundle defaults to the 64 bit architecture. in case you want to bundle 32 bit OS use the -r i386 switch to the bundle command. First, bundle the image.

input this command:
# euca-bundle-image -i fedoraV11.10.x86-64.img --kernel $EKI --ramdisk $ERI
 
 
 
and you will get this in reply:
Checking image
Tarring image
Encrypting image
Splitting image...
Part: fedoraV11.10.x86-64.img.part.0
------Clip-----
Part: fedoraV11.10.x86-64.img.part.16
Generating manifest /tmp/fedoraV11.10.x86-64.img.manifest.xml
LikeMadhiyan, Jagadeesan (EXT-Wipro Technologies - IN) likes this
sEdit Labels
1 Comment
User icon: kbarczyn
Barczynski, Krzysztof (NSN - US/Mountain View)
Nice article, I have been playing around getting custom image. Everytime instance fails to start, here are the steps:

1. create ebs backed instance from image emi-BAF043AA (143382742018/cfw-image-centos-6-3-64bit-12)

euca-run-instances  -k sqlsESPO6.key -g sqlserver -z escloc06_2 -n 1  -t m1.large emi-BAF043AA
Instance launches, I run chef solo to configure it - all works fine. 

Since I did not specify the kernel during lunch time, I lookup kernel with:

euca-describe-instances i-AACF45A6
RESERVATION r-7F024318  247345555413    sqlserver
INSTANCE    i-AACF45A6  emi-BAF043AA    euca-10-159-11-168.eucalyptus.escloc06.eecloud.nsn-net.net  euca-10-254-100-252.eucalyptus.internal running sqlsESPO6.key   0       m1.large    2012-12-04T23:21:08.758Z    escloc06_2  eki-CB423582            monitoring-disabled euca-10-159-11-168.eucalyptus.escloc06.eecloud.nsn-net.net  euca-10-254-100-252.eucalyptus.internal         ebs
Voila: eki-CB423582.

Q1: As ramdisk was not specified none is used?

After installing tons of stuff I am ready to create image.

2. Create&attach ebs volume:

euca-create-volume -s 15 -z escloc06_2
VOLUME     vol-CCC33B4F     15     escloc06_2     creating     2012-12-05T00:39:13.309Z
euca-attach-volume -i i-AACF45A6 -d /dev/sdg  vol-CCC33B4F
ssh euca-10-159-11-168.eucalyptus.escloc06.eecloud.nsn-net.net
fdisk -l #check device name
mkfs.ext4 /dev/vdc
mount /dev/vdc  /ebs/
Q? is ext4 ok - or should I use ext3?
3. Get&configure euce2ools

yum install euca2ools
cat ~/.eucarc
EC2_PRIVATE_KEY=
EC2_ACCESS_KEY=
EC2_SECRET_KEY=
EC2_URL=http://10.159.11.29:8773/services/Eucalyptus
4. Set s3 url:
export S3_URL=http://esclos117.emea.nsn-net.net:8773/services/Walrus/
5. Certificate ?

EUCALYPTUS_CERT=/etc/cert
EC2_CERT=/etc/cert
How should I obtain certificate - my guess is management console (espo6): https://escloc06.emea.nsn-net.net:8443/
After login and selecting "Certificates" I saw three certificates - each is active, so I picked random one.
Then copied PEM to my instance to /etc/cert.

Q2: Should I exclude that file - when bundling instance?

6. Bundling:
I got exception about missing variable EC2_USER_ID, so I set it to the user name I use for EECloud (kb...).

euca-bundle-vol -p ret21 -d /ebs -s 9216 --exclude /ebs/,/var/run/dhclient-eth0.pid,/root/.eucarc --no-inherit --kernel eki-CB423582
7. Uploading bundle:

euca-upload-bundle -b $BUCKETNAME -m /ebs/ret21.manifest.xml
8. Register image:

euca-register  $BUCKETNAME/ret21.manifest.xml -n retina21
IMAGE  emi-XXXXXXX
9. Running instance:

euca-run-instances  -k sqlsESPO6.key -g sqlserver -z escloc06_2 -n 1  -t m1.large emi-XXXXXX
After 20s instance status goes from pending to shutdown, then terminated.

Any hints? I have repeated that process over 20 times with different parameters but non of them succeeded.