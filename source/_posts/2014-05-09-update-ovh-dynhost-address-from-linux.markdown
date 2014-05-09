---
layout: post
title: "Update OVH DynHost address from Linux"
date: 2014-05-09 21:45:39 +0200
comments: true
categories: Hack
---

I have my domain name registered at [OVH][ovh]. To update a DNS address from a machine behind a dynamic IP, they provide a feature called **DynHost**. DynHost uses the same protocol as DynDNS for updating their website.

From your DNS Section of the [OVH manager webapp][ovh-manager], you can create a DynHost account: you will have to specify the DNS address you want to update, a username and password for this account.

**Warning**: The script that update your DynHost address sends your login and password uncrypted! Don't use the same login/password as your main OVH Manager account.

Once created, you can configure your home server to update your DynHost.

Configure Updatedd
------------------

Updatedd is an utility recommended by OVH to update your address from Linux. You can find it [here][updatedd].

<!-- More -->

Get the sources:

{% codeblock lang:console %}
~$ wget http://nongnu.askapache.com/updatedd/updatedd_2.6.tar.gz
{% endcodeblock %}

Then, you will have to fix an error in a configuration file before compile it.
Extract and edit the libovh file:
{% codeblock lang:console %}
~$ tar xvf updatedd_2.6.tar.gz
~$ nano updatedd-2.6/src/plugins/libovh.h
{% endcodeblock %}

And replace the host **ovh.com** on line 24 by **www.ovh.com**. You should have:
{% codeblock %}
define DYNDNSHOST "www.ovh.com"
{% endcodeblock %}

We can now compile updatedd:
{% codeblock lang:console %}
~$ ./configure
~$ make
~$ sudo make install
{% endcodeblock %}

You can now use updatedd to update your domain name with the command:
{% codeblock lang:console %}
updatedd ovh -- --ipv4 yourIP dynHostUsername:dynHostPassword host
{% endcodeblock %} 

Automate the DynHost Update
---------------------------

To update your account, create a script and run it on a daily base with cron.

Create a Script with this content:

{% codeblock lang:bash %}
#!/bin/bash

## dynhost parameters
username=dynHostUser
password=dynHostPassword
host=yourmain.com

##Log (1=true,0=false)
log_change=1
log_no_change=0
log_file=/var/log/dynhost.log

#File with old IP
old_ip_file=/var/cache/ip_old

touch ${old_ip_file}
touch ${log_file}

#Get public IP
ip="`dig +short myip.opendns.com @resolver1.opendns.com`"
ip_old=`cat ${old_ip_file}`

#Compare IP
if [ "${ip}" = "${ip_old}" ]
then 
   if [ "${log_no_change}" = "1" ]
   then
      echo `date`: No IP change was found >> ${log_file}
   fi
else
   echo ${ip} > ${old_ip_file}
   if [ "${log_change}" = "1" ]
   then
      echo "`date`:IP has changed. (Old : ${ip_old}, New : ${ip})" >> ${log_file}
      updatedd ovh -- --ipv4 ${ip} ${username}:${password} ${host} >> ${log_file}
   else
      updatedd ovh -- --ipv4 ${ip} ${username}:${password} ${host}
   fi
fi

{% endcodeblock %}

Don't forget to change DynHost parameters with yours in the script.

Make the script executable and put both updatedd and your script in a folder in your path. For instance:

{% codeblock lang:console %}
~$ chmod +x yourscript 
~$ cp updatedd yourscript /usr/local/bin

{% endcodeblock %}

Finally, put into your cron so that it is executed on a daily base (here every 30 minutes):

{% codeblock lang:bash %}
sudo echo "30  *    * * *   root    your-script" >> /etc/crontab
{% endcodeblock %}

The steps and scripts described here have been adapted from [this article][lermit].
This steps should work for any provider supporting DynDNS protocol.

[ovh]: http://www.ovh.com
[ovh-manager]: https://www.ovh.com/managerv3
[updatedd]: http://nongnu.askapache.com/updatedd/updatedd_2.6.tar.gz
[lermit]: http://lermit-informatique.blogspot.de/2009/08/ovh-le-dynhost-de-ovh-et-updatedd.html
