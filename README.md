# xinetd #

[![Build Status](https://travis-ci.org/openSUSE/xinetd.svg?branch=master)](https://travis-ci.org/openSUSE/xinetd)

xinetd is a powerful replacement for inetd.
Original site: http://www.xinetd.org
xinetd has access control mechanisms, extensive logging capabilities,
the ability to make services available based on time, can place
limits on the number of servers that can be started, and has deployable
defence mechanisms to protect against port scanners, among other things.

There are a number of differences between xinetd and inetd.  The 
largest difference to the end user is the config file.  xinetd's 
config file format is more C like, and somewhat similar to bind 8's.

*** NOTE ***
xinetd's reconfig signal is now SIGHUP.  SIGUSR2 still works for now,
but should be considered deprecated.  The state dump signal is now
SIGUSR1.
***      ***

# There are a couple of compile time options to xinetd

--with-libwrap  : This option to the configure script tells xinetd
to compile in support for tcp wrappers.  You must already have libwrap
installed on your system.  This option will have xinetd pay attention
to your /etc/hosts.{allow|deny} files.  With this option turned on,
xinetd will first look at your /etc/hosts.{allow|deny} files, then
if access is granted, it goes through xinetd's internal access control
mechanisms.  Note that xinetd passes the server name if there is one,
otherwise it uses the service id, *not* the service name to libwrap - 
this is a change from previous behaviour.

--with-loadavg  : This option to the configure script tells xinetd
to compile in support for the max_load configuration option.  This
option allows you to have certain services disabled when the system
load gets above a specified level.

--with-howl: Adds howl mdns advertising support to xinetd.

# Access Control

As of xinetd 2.1.8.8pre3 there is a change in the handling of 
names specified in the access control directives, only_from and
no_access.  For numerical entries in these fields, nothing has
changed.

The way it used to be:  When xinetd started, it would lookup the
name(s) specified.  It would then collect all the IP addresses
associated with that name, and store them in the access control
lists.  This led to problems of acls being out of date, and 
access controls being messed up when names were specified.  It did
have the advantage of not waiting for a lookup to happen when you
connected.

The way it is now:  xinetd keeps all the names you specify on the
access control directives.  When a client attempts to connect to
a service, a reverse lookup is performed on the client's IP address.
The canonical name returned is compared with the specified names.
If the first character of the name being specified in the config
file is a '.', then all hosts within that domain are matched.
For example, if I put .synack.net, all hosts with a reverse mapping
that are in .synack.net domain, are matched.

# libwrap support

For libwrap access control, the access control is done by the
server name for the service.  So, if you have an entry like this:
service telnet
{
     ...
	 server = /usr/sbin/in.telnetd
	 ...
}
Your corresponding hosts.{allow|deny} entry would look something
like this:
in.telnetd: ALL

However, many services don't have a "server".  Internal services
and redirection services don't have a "server" line in the configuration
file.  For these services, the service name is used.  For example:
server telnet
{
	...
	redirect = 10.0.0.1 23
	...
}
Your hosts.{allow|deny} entry would look something like this:
telnet: ALL

So, in general, if a service has a "server" attribute to it, access
control is performed based on that entry.  If a service does not have
a "server" attribute, (internal and redirection services) then access
control is based on the service name.
This is only for libwrap access control.

# itox

I will continue to keep itox in the distribution, and fix any bugs or
compatibility issues that come to my attention, but I probably won't
get around to adding features.  If someone else would like to pick this
up, let me know and I'd be happy to incorporate changes.
I have made a quick perl script (xconv.pl) to replace itox.  xconv.pl
does a straight translation of inetd.conf file syntax to xinetd.conf.
It handles tcpd correctly, warns on the use of rpc services, 
sets the REUSE flag, and handles groups properly if your inetd.conf file 
supports them.

xinetd was originally written by panos@cs.colorado.edu.  At least one other
version of xinetd has been seen floating around the net.  This version is
being maintained by Rob Braun (bbraun@synack.net) and bug reports for this
version should be directed there.

Known bugs: On some BSD's, you may need to set 'groups = yes' in your services.
If you need this, and you don't specify 'groups = yes', you'll get a log 
message telling you it is needed, when you connect to a service.

Bug reports/comments/suggestions/flames for this version should be sent
to bbraun@synack.net
