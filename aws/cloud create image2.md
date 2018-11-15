Example 1.

[root@esclos05 netact]# parted netact-rhel58-v1.img unit s print
Model:  (file)
Disk /opt/isostore/tiahokas/netact/netact-rhel58-v1.img: 16777216s
Sector size (logical/physical): 512B/512B
Partition Table: msdos
 
Number  Start  End        Size       Type     File system  Flags
 1      63s    16771859s  16771797s  primary  ext3         boot
 
[root@esclos05 netact]# time dd if=netact-rhel58-v1.img of=netact-rhel58-v1-stripped2.img bs=512 skip=63 count=16771797
16771797+0 records in
16771797+0 records out
8587160064 bytes (8.6 GB) copied, 140.822 s, 61.0 MB/s
 
real    2m20.856s
user    0m1.015s
sys     0m25.629s
You can obtain all of the necessary values for the dd command directly from the parted output:

bs = Sector size
skip = Start
count = Size
Example 2.

(parted) p
 
Model:  (file)
Disk /opt/linsee54_64/esclos09v1.img: 10485759999B
Sector size (logical/physical): 512B/512B
Partition Table: msdos
 
Number  Start        End           Size         Type     File system  Flags
 1      32256B       9533099519B   9533067264B  primary  ext3         boot
 2      9533099520B  10479006719B  945907200B   primary  linux-swap
Using block size 512 to speed things up

Skip from start Start 32256 -> 32256/512 = 63
continue 9533067264/512 = 18619272

[EE:root@esclos03 linsee54_64]# time dd if=esclos09v1.img of=rhel54v164b bs=512 skip=63 count=18619272
18619272+0 records in
18619272+0 records out
9533067264 bytes (9.5 GB) copied, 33.8087 seconds, 282 MB/s
real    0m33.810s
8. Upload to eucalyptus

After you have an rootfs image you can upload and register it. The upload process consists of three separated steps

euca-bundle-image (creates the uploadable bundle and splits it into chunks)
euca-upload-bundle (uploads the bundle to the cloud) 
euca-register (registers an uploaded bundle)
cd /tmp
rm -rf <previous image>
Example

euca-bundle-image -i vmlinuz-2.6.18-194.el5 --kernel true
euca-upload-bundle -b rhel55kerbucket -m /tmp/vmlinuz-2.6.18-194.el5.manifest.xml
euca-register -d 'Description of Kernel' rhel55Kerbucket/vmlinuz-2.6.18-194.el5.manifest.xml
export EKI=eki-F17714CE
 
euca-bundle-image -i initrd-2.6.18-194.el5.img --ramdisk true
euca-upload-bundle -b rhel55rambucket -m /tmp/initrd-2.6.18-194.el5.img.manifest.xml
euca-register -d 'Description of Ramdisk' rhel55rambucket/initrd-2.6.18-194.el5.img.manifest.xml
export ERI=eri-2BD115B6
 
euca-bundle-image -i rhel55v3_new2.img --kernel $EKI --ramdisk $ERI
euca-upload-bundle -b rhel55bucket -m /tmp/rhel55v3_new2.img.manifest.xml
euca-register -d 'Description of Image' rhel55bucket/rhel55v3_new2.img.manifest.xml
If you have made it this far it might be good idea to read about this from internet.

http://open.eucalyptus.com/participate/wiki/creating-images-iso-kvm

Userdata
(At least) In Eucalyptus 3.0.1, the user data service returns 404, if you query it too fast. Curl, by default, considers this a successful request and does not retry afterwards. You can use the attached robustcurl.sh script which handles 404 errors better than the stock curl.

For example, if you have the following line in /etc/rc.local:curl --retry 3 --retry-delay 10 -m 45 -s http://169.254.169.254/latest/user-data/ >> /root/.userdata

If the user data service returns a 404 error, then your .userdata file will be the error HTML dump. You can replace the above line with:

robustcurl.sh [http://169.254.169.254/latest/user-data/] >> /root/.userdata
Place the robustcurl.sh script into /usr/bin on the image you are creating. Remember to give it execute rights with chmod a+x robustcurl.sh

The script does not accept any other parameters or options than the URL, so it is not a drop-in replacement for curl. If you place the robustcurl.sh script to your image, place also the execute line on rc.local file:

. /root/.userdata

Otherwise the userdata is only recorded to root/.userdata file but will not be executed.

Here?s the robustcurl.sh

#!/bin/sh
 
# Usage: robustcurl.sh URL
 
TIMEOUT=45
DELAY=15
RETRIES=10
OUTFILE=/tmp/curlfile.$$
 
for (( i=0; i < RETRIES; i++ )); do
    curl -sf -m $TIMEOUT $1 > $OUTFILE.$i
    ERROR=$?
    if [ "$ERROR" -eq 0 ]; then
        cat $OUTFILE.$i
        exit 0
    fi
    echo Curl error code $ERROR, attempt $i >> /tmp/errors.$$
    sleep $DELAY
done