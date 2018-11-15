# swarm plugin #
1. can find the jenkins master automaticlly and connect
2. can be run in  linux and master
3. cannot auto create from jenkins master

# disk monitor plugin #
1. it will take too many system resources, cannot install 

# copy artifacts plugin #
1. find the  jars  base on the  fingerprint 
 if the fingerprint info is error, the it will copy error jars
 cannot write verid  characters , otherwise it will copy error jars ,(the latest release has fixed this problem )


# ec2 plugin #

1. can auto create amazon slaves in jenkins
2. if we add 2 cloud server in jenkins , it will use the first one to launch slave instance, 
	mainly function, just monitor the jenkins  asyncwork  
3. auto delete slave instances has a bug ,(not sure it is fixed in the latest release)


# svn plugin #
run svn polling in slave, when the network is not stable, it will be blocked .
resolution:
1.delete the slave
2.restart jenkins
3. disconnect slave
for more info   JENKINS_URL/descriptor/hudson.triggers.SCMTrigger/


    
# jenkins jobConfigHistory plugin #
it will update the job config file. even if it is changed by scripts