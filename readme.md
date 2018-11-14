# Cppcms configure with Lighttpd

This guide is a simple walkthrough on how to configure cppcms and lighttpd on a FreeBSD server

> OS: FreeBSD 11.2
> 
> Cppcms: cppcms 1.2.1
> 
> Lighttpd: 1.4.51

### Prerequisites
- A server running FreeBSD 11.0.
- A user account configured to run commands with sudo. The default freebsd account that comes with a Vultr FreeBSD will be fine for this tutorial. 

## Step 1 — Installing Lighttpd
I will be using pkg tools to install lighttpd in here for ease.

To install Lighttpd with its package, first update the repository information to ensure you have the latest list of available packages:

`$ sudo pkg update `

Next, download and install the **lighttpd** package:

`$ sudo pkg install lighttpd`

Confirm the installation by typing y. Lighttpd will install.

### Disable Ipv6 from default configuration (Optional)
This is because the default Lighttpd configuration isn't completely configured to support IPv6. To avoid surprises later, edit Lighttpd's configuration file and disable support for IPv6, since you won't need it to complete this tutorial. You can enable it in the future if you decide to use it:

`$ sudo ee /usr/local/etc/lighttpd/lighttpd.conf`

Locate this section:

>...
>
>##
>
>## Use IPv6?
>
>##
>
>server.use-ipv6 = "enable"
>
>...

Change enable to disable:

>...
>
>...
>
>server.use-ipv6 = "disable"
>
>...

Next, locate this line at the very end of the configuration file:

>...
>
>...
>
>$SERVER["socket"] == "0.0.0.0:80" { }

Comment it out, as it's unnecessary when we're not using IPv6:

>...
>
>...
>
>#$SERVER["socket"] == "0.0.0.0:80" { }

Then save the file and exit the editor.




## Step 2 — Installing Cppcms

Install Cmake

`$ pkg install cmake`

Configure Portsnap to install Cppcms

The base system of FreeBSD includes Portsnap. This is a fast and user-friendly tool for retrieving the Ports Collection and is the recommended choice for most users. This utility connects to a FreeBSD site, verifies the secure key, and downloads a new copy of the Ports Collection. The key is used to verify the integrity of all downloaded files.

To download a compressed snapshot of the Ports Collection into /var/db/portsnap:

`$  portsnap fetch`

When running Portsnap for the first time, extract the snapshot into /usr/ports:

`$ portsnap extract`

After the first use of Portsnap has been completed as shown above, /usr/ports can be updated as needed by running:

`$ portsnap fetch update`

Next, install the [Cppcms Ports](https://www.freshports.org/www/cppcms/)

`$ cd /usr/ports/www/cppcms/ && make install clean`

Accept and install cppcms
`yes`

Next, install the dependency, `yes to all`

## Step 3 - Connecting everything

### Configure Cppcms and Build

In all this guide we assume:
- Application's executable placed in /opt/app/bin/hello
- Application's configuration file placed in /opt/app/etc/config.js
- Our application's URL (script) is /hello

For your reference, [Cppcms official guide](http://cppcms.com/wikipp/en/page/cppcms_1x_tut_web_server_config)

In this guide, I will be using filezilla sftp for my case to upload file to server.

Copy opt folder in this github repo to your server, it contains a simple hello world cppcms project.

Folder structure
>/opt
>>App
>>>Bin
>>>>Executable project 

>>>Etc  
>>>>Configuration file

Go to bin directory, 

`$ rm hello`

`$ make`




### Configure lighttpd

`$ sudo ee /usr/local/etc/lighttpd/conf.d/fastcgi.conf`

Locate the following section:

>...
>
>\##
>
>\## FastCGI (mod_fastcgi)
>
>\##
>
>\#include "conf.d/fastcgi.conf"
>
>...

and change to the following block,

`Uncomment include`

>...
>
>\##
>
>\## FastCGI (mod_fastcgi)
>
>\##
>
>include "conf.d/fastcgi.conf"
>
>...

Save the file and exit editor

Next, edit the FastCGI config file:

`$ sudo ee 
/usr/local/etc/lighttpd/conf.d/fastcgi.conf`

Paste this block according to your build file

>fastcgi.server = (   
>  "/hello" => ((  
    ## Command line to run  
    "bin-path" => "/opt/app/bin/hello -c /opt/app/etc/config.js",  
    "socket" => "/tmp/hello-fcgi-socket",  
    ## Important - only one process should start  
    "max-procs" => 1,   
    "check-local" => "disable"  
  ))  
)  


`$ sysrc lighttpd_enable=yes`

`$ service lighttpd start`

## Voila, done
goto \<your ip address>/hello in browser to see the result

## Additional Step for routing throught cloudflare
1. Cloudflare
2. Add domain
3. Change name server for your domain 
4. Go into dns record (Could take up to 48 hours)
5. Add A name record, Host : @, Ip Address : Vultr Ip Address