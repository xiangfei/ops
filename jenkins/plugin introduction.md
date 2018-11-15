# swarm plugin #
1. can find the jenkins master automaticlly and connect
2. can be run in  linux and master
3. cannot auto create from jenkins master

# disk monitor plugin #
1. it will take too many system resources, cannot install 

# jenkins copyartifacts plugin #

1. find the  jars  base on the  fingerprint 
 if the fingerprint info is error, the it will copy error jars
 cannot write verid  characters , otherwise it will copy error jars ,(the latest release has fixed this problem )

# jenkins ec2 plugin #

1. can auto create amazon slaves in jenkins
2. if we add 2 cloud server in jenkins , it will use the first one to launch slave instance, 
	mainly function, just monitor the jenkins  asyncwork  
3. auto delete slave instances has a bug ,(not sure it is fixed in the latest release)

# jenkins svn plugin #

run svn polling in slave, when the network is not stable, it will be blocked .
resolution:
1.delete the slave
2.restart jenkins
3. disconnect slave
for more info   JENKINS_URL/descriptor/hudson.triggers.SCMTrigger/

#  jenkins  jobConfigHistory  plugin #

it will update the job config file. even if it is changed by scripts

# plugin list #
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  3448802 Jan 16  2013 all-changes.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   183785 Dec 16  2013 analysis-collector.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  2854140 Dec 16  2013 analysis-core.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   112483 Aug 30 10:21 antisamy-markup-formatter.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    90421 Aug 30 10:21 ant.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    15843 Apr  3  2013 any-buildstep.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    54204 Aug 30 10:58 authorize-project.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1762584 Dec 16  2013 build-failure-analyzer.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    24640 Jan 16  2013 build-metrics.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing     9611 Apr 10  2013 build-name-setter.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1660603 Dec 20  2012 build-pipeline-plugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    33010 Dec 16  2013 build-timeout.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  2372360 Dec 16  2013 checkstyle.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    36810 Dec 16  2013 conditional-buildstep.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   673694 Dec 16  2013 configurationslicing.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  2927585 Dec 16  2013 copyartifact.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing     6528 Jun  9  2013 countjobs-viewstabbar.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   371034 Aug  4 07:55 credentials.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   929025 Dec 16  2013 cvs.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   599851 Dec 16  2013 dashboard-view.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  5783882 Oct  8  2013 database.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   529982 Oct  8  2013 database-postgresql.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  5134796 Dec 16  2013 depgraph-view.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    16247 Apr 10  2013 description-setter.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   307117 Jan 14  2014 disk-usage.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    15305 Jun 13  2013 display-upstream-changes.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  3432949 Dec 30  2013 ec2.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing 11033842 Dec 16  2013 email-ext.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   376251 Dec 10  2012 emmacoveragecolumn.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1341631 Dec 10  2012 emma.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    15211 Nov 18  2013 environment-script.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    53839 Dec 16  2013 external-monitor-job.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  8091961 Dec 16  2013 findbugs.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    48479 Dec 16  2013 flexible-publish.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  2382882 Dec 16  2013 git-client.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  4827296 Dec 16  2013 git.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   336716 Sep 16 16:36 global-build-stats.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    43532 Dec 16  2013 groovy.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    21350 Dec 16  2013 htmlpublisher.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    37364 Feb  6  2013 hudson-pview-plugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    10231 Jan 24  2013 hudson-wsclean-plugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  4804046 Dec 16  2013 jacoco.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    38073 Aug 30 10:21 javadoc.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   158796 Dec 16  2013 jobConfigHistory.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1672427 Jan 17  2013 jquery.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    91202 Jan 17  2013 jquery-ui.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   321735 Aug 30 10:21 junit.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  6424714 Aug 19 18:13 klocwork.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    48767 Dec 16  2013 ldap.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    23790 Feb  6  2013 locks-and-latches.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    45518 May 31  2013 log-parser.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   113475 Dec 16  2013 mailer.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    79681 Aug 30 10:21 matrix-auth.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   237973 Aug 30 10:21 matrix-project.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing 10773666 Aug 30 10:21 maven-plugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  2901505 Dec 16  2013 monitoring.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    30626 Apr  3  2013 mttr.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    28584 Jul 23  2013 nested-view.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    70637 Dec 16  2013 nodelabelparameter.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1030550 Dec 16  2013 pam-auth.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1207752 Dec 16  2013 parameterized-trigger.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1196223 Dec 18  2013 postbuildscript.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    12556 Jan  8  2013 PrioritySorter.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    24920 Apr 10  2013 project-description-setter.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    18022 Jan 10  2013 project-stats-plugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  3414900 Oct  9  2012 promoted-builds-simple.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    13135 Mar 27  2013 purge-build-queue-plugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    45124 Dec 16  2013 rebuild.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    37198 Nov 18  2013 RessuPlugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   145650 Dec 16  2013 run-condition.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    75264 Dec 16  2013 scm-api.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    36275 Oct 16  2013 SCTPlugin.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    79525 Dec 16  2013 sectioned-view.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing     6932 Dec 20  2012 show-build-parameters.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   407773 Dec 16  2013 sonar.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    79972 Aug  4 07:55 ssh-credentials.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    66901 Aug  4 07:55 ssh-slaves.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  4683101 Dec 16  2013 subversion.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    25730 Dec 16  2013 throttle-concurrents.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    45091 Dec 16  2013 timestamper.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    23388 Dec 16  2013 token-macro.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    50065 Jan  7  2013 translation.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   313873 Dec 16  2013 view-job-filters.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   990106 May 31  2013 violations.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  1251030 Dec 16  2013 warnings.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing   949638 Aug 30 10:21 windows-slaves.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    27562 Dec 16  2013 ws-cleanup.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing  2138864 May 16 18:37 xunit.jpi
-rw-r--r-- 1 uicaci linseeusers_dhn_beijing    21882 Dec 16  2013 xvnc.jpi