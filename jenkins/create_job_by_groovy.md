// create project 

import  jekins.modle.*
import hudson.tasks.*
  
jenkins = Jenkins.instance

def property='''
job_root=${WORKSPACE}
installer_name=${installer_name}
branch_name=${branch_name}
force_build_version=${force_build_version}
force_validator_version=${force_validator_version}'''
def buildfile='''
${WORKSPACE}/scripts/trsmanager_installer/new-trsmanager-installer.xml 
'''

FreeStyleProject  project =  new FreeStyleProject(jenkins, "fei_free_styleproject")
Ant  ant = new Ant("trsmanager_installer_zip","Ant_1.9.4" ,null , buildfile , property)
project.buildersList.add(ant)
project.save()
jenkins.putItem(project)