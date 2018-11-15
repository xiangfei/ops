import hudons.model.*

import jenkins.model.*

import hudson.remoting.*

 

Jenkins.instance.getTrigger("SCMTrigger").getRunners().each()

{

  item ->

  println(item.getTarget().name)

  println(item.getDuration())

  println(item.getStartTime())

  long millis = Calendar.instance.time.time - item.getStartTime()

 

  if(millis > (1000 * 60 * 30))

  {

    Thread.getAllStackTraces().keySet().each()

    {

      tItem ->

      if (tItem.getName().contains("SCM polling") && tItem.getName().contains(item.getTarget().name))

      {

        println "Interrupting thread " + tItem.getName();

        tItem.interrupt()

      }

    }

  }

}