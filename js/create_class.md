<!DOCTYPE html>
<html>
<head>
<script>
//this is a define public fields
function myclass() {
  this.propertytwo = 2;
  this.propertythree = 3;
}
myclass.prototype.propertyone = 1;
myclass.prototype.propertytwo = 2;

var myinstance = new myclass();
myinstance.propertyfour = 4;

alert(myinstance.hasOwnProperty('propertyone'));
//alerts false

alert(myinstance.hasOwnProperty('propertytwo'));
</script>
</head>

<body>
</body>

</html>


define filed in private 
function myclass() {
  this.propertytwo = 2;
  var   propertythree = 3;
}