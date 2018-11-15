1.auto update ant version  in jobs

import hudson.model.*
import hudson.plugins.promoted_builds.*
import hudson.plugins.promoted_builds.conditions.*
import hudson.tasks.*
import hudson.plugins.parameterizedtrigger.*
def property='''
job_root=${WORKSPACE}
installer_name=${installer_name}
branch_name=${branch_name}
force_build_version=${force_build_version}
force_validator_version=${force_validator_version}'''
def buildfile='''
${WORKSPACE}/scripts/trsmanager_installer/new-trsmanager-installer.xml 
'''
def jenkins = hudson.model.Hudson.instance
FreeStyleProject item= jenkins.getItem("UI.Installer_trsmanager_LNZ5.0")
for(publisher in item.publishersList) { 
  println publisher
    }
// change  Ant Version
for(builder  in  item.buildersList){
  // println builder
     
     if  (builder  instanceof  hudson.tasks.Ant){
       item.buildersList.remove(builder)
        // builder.antName="Ant_1.9.4"
      Ant  ant = new Ant("trsmanager_installer_zip","Ant_1.9.4" ,null , buildfile , property)
      item.buildersList.add(ant)   
         
     }
 }       
 item.save()