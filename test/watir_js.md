require 'watir'
b = Watir::Browser.start "http://somepagewithdialogs"
# don't return anything for alert
b.execute_script("window.alert = function() {}")

# return some string for prompt to simulate user entering it
b.execute_script("window.prompt = function() {return 'my name'}")

# return null for prompt to simulate clicking Cancel
b.execute_script("window.prompt = function() {return null}")

# return true for confirm to simulate clicking OK
b.execute_script("window.confirm = function() {return true}")

# return false for confirm to simulate clicking Cancel
b.execute_script("window.confirm = function() {return false}")

# interact with some element which would trigger the pop up
b.button(:id => "dialogTrigger").click




# deal with js 2 #



def startClicker( button , waitTime = 3)
   w = WinClicker.new
   longName = $ie.dir.gsub("/" , "\\" )
   shortName = w.getShortFileName(longName)
   c = "start ruby #{shortName}\\watir\\clickJSDialog.rb #{button} #{waitTime} "
   puts "Starting #{c}"
   w.winsystem(c)
   w=nil
 end

#Then call it right before you click the button (or whatever) that
causes the javascript popup to display:

startClicker("OK" , 3)
$ie.button("Submit").click