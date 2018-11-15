import hudson.model.*
//gets current promotion build env variables
def thisbuild = Thread.currentThread().executable  
def envVars= thisbuild.properties.get("envVars"); 

//finally we can set the build description
build.setDescription(envVars["product_name"]);