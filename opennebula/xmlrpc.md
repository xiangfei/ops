# ruby xml prc #
```
require 'xmlrpc/client'
server = XMLRPC::Client.new("192.168.65.10", "/RPC2", 2633)
server.call("one.system.version", "oneadmin:awifi@123")
```

> java ruby sdk  ,command line 需要在master机器运行 
