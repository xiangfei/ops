1. it is a restrict jenkins job , has its own config.xml file
2. it is hard to use auoto create jenkins jobs from  parameters 
3. if we don't add the  permission control , the jobs can be triggered by anyone .

import hudson.model.*
import hudson.plugins.promoted_builds.*
import hudson.plugins.promoted_builds.conditions.*
import hudson.tasks.*
import hudson.plugins.parameterizedtrigger.*  
def jenkins = hudson.model.Hudson.instance
FreeStyleProject job= jenkins.getItem("UI.Installer_trsmanager_LNZ4.0")
JobPropertyImpl promotion = new JobPropertyImpl(job)
job.addProperty(promotion);  
PromotionProcess proc = promotion.addProcess("Ready For Release")
proc.icon = "star-blue"
proc.conditions.add(new DownstreamPassCondition("Fei_Test_Groovy"))
proc.conditions.add(new ManualCondition())
proc.getBuildSteps().add(new Shell("ls"))
proc.save() 
//proc.scheduleBuild()  
JobPropertyImpl promotion2 = new JobPropertyImpl(job)
job.addProperty(promotion);  
PromotionProcess proc2 = promotion.addProcess("Release Build Test")
proc2.icon = "star-green"
proc2.conditions.add(new DownstreamPassCondition("Fei_Test_Groovy"))
BuildTriggerConfig  triggerconfig =new BuildTriggerConfig("Fei_Test_Groovy", ResultCondition.UNSTABLE_OR_BETTER, new PredefinedBuildParameters("SCM=HZ/nmember=feixiang"))    
BuildTrigger  parametrigger =  new BuildTrigger(triggerconfig)  
proc.getBuildSteps().add(parametrigger)
proc2.save()
job.save()