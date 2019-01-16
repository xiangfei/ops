It occurs somewhat often on Gamling.org. I'm still trying to figure out what really happens.

I have found on the log and this is the error message:

unix:/passenger_helper_server failed (11: Resource temporarily unavailable) while connecting to upstream

The solutions I have tried

1.
One proposed solution is to increase the buffer size for header.
(http://stackoverflow.com/questions/2307231/how-to-avoid-nginx-upstream-sent-too-big-header-errors)

It does not seem to solve the problem.

2.
The second solution is to disable firewall. It does not solve the problem.


3.
This is the one that actually solves the problem.

The problem is that the backlog queue (for imcoming connection) is full. Therefore, we shall increase it.

The default value of net.core.somaxconn is 128.

I have increased it to 40,000! with this command, sysctl -w net.core.somaxconn=40000.

The default value of net.core.netdev_max_backlog is 1000. I have increased it to 40,000 with this command sysctl -w net.core.netdev_max_backlog=40000.

Also do not forget to set open file limits:

just increase it to 100,000 with ulimit -n 100000

And edit this file /etc/security/limits.conf


That does seem to solve problems just a little.

The real problem is that when Passenger spawns a new instance, it clones a previous process. It also clones THE SOCKET FILE.