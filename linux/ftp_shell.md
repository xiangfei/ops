USER=userid
PASSWD=userpw
ftp -n f2dev <<SCRIPT
user $USER $PASSWD
binary
get some.file
quit
SCRIPT