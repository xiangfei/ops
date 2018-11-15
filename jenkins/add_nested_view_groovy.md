import  jenkins.model.* 
import  hudson.model.*
import  hudson.plugins.nested_view.*
NestedView view = new NestedView("Fei_Test_Nested_View")
view.setOwner(Jenkins.instance.getView("All_Products").getOwner())
ListView listview =  new ListView("List_View",view)
listview.add(Hudson.instance.getItem("Fei_Test_Groovy"))
view.addView(listview)
Jenkins.instance.addView(view)