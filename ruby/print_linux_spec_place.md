#!/usr/bin/env ruby
system ("clear")
while true   do

#`clear`
#system ("clear")
sleep 1
xxx=rand(1000)
puts  `tput cup 0 4`
puts  "value/seconds          statistic_name\n"
puts `tput cup 1 4`
puts "--------------- ---------------------------------------------------------------\n"
puts "this is  #{xxx}       statistic_name    "
end