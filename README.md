# raspberrypi-get-ip

Have you ever been annoyed that every time you reboot your Raspberry Pi (or take it to a different network) you have to guess its IP address so that you can ssh into it? No? Well, I have, and that's why I've created this tutorial.

Here, we'll set your Raspberry Pi up to send you an email with its IP address every time it boots using [Gmail](https://gmail.com).

**Note:** If you feel like this is an overkill, you can always use `nmap`, for example:

```
$ nmap -sP 192.168.1.1/24
```

## Install and configure ssmtp

This section is partly based on this [cyberciti.biz blog post](https://www.cyberciti.biz/tips/linux-use-gmail-as-a-smarthost.html) (accessed on 18-01-2020).

```
$ apt-get update && apt-get install ssmtp
```

Now, configure gmail as a smarthost:

```
$ vi /etc/ssmtp/ssmtp.conf
```

Update this file with the following settings:

```
AuthUser=you@gmail.com
AuthPass=Your-Gmail-Password
FromLineOverride=YES
mailhub=smtp.gmail.com:587
UseSTARTTLS=YES
```

Test it:

```
$ echo "Hello world!" | ssmtp you@example.com
```

**Note:** You might bump into authentication problems on your first try. In my case, Gmail sent me an email alerting that a less secure application was trying to log into my account. I followed the links in the message and changed settings to allow ssmtp to log in. I also had to enable IMAP access on my Gmail settings.

## Create the email sending script

Create a file named `mail_ip.sh` (or whatever you prefer) and edit it adding your email address:

```
mailto="you@example.com"
ip=`ip route list | awk '{print NR,$(NF-2)}'`

{
	echo To: $mailto
       	echo Subject: [RasPi] My IP
	echo "$ip"
} | /usr/sbin/ssmtp $mailto
echo "$ip"
echo "Finished running at `date`"
```

Then, make sure it is executable and try to run it:

```
$ chmod +x mail_ip.sh
$ ./mail_ip.sh
```

If everything goes well, you should receive an email like this:

```
1 192.168.1.46
2 192.168.1.46
```

## Set your script to run on reboot

There are different ways to do this, we'll use `crontab` here.

```
$ crontab -e
```

Add the following line to the end of the file (make sure to use the absolute path to the script):

```
@reboot sleep 120 && /home/pi/mail_ip.sh > /home/pi/mail_ip.log 2>&1
```

**Note:** In my case, I had to add a 2-minute sleep before running it, so that ssmtp had time to start up. You might need a different delay, or none at all (it took me a while of trial and error to figure this out). Redirecting stdout and stderr to a log file is not necessary, but it helped me debug it.

You can also add `MAILTO=you@example.com` at the top of the crontab file to make sure you'll receive notifications when cron jobs fail.

## Try it out!

Either run `$ sudo reboot` or unplug and plug your Raspberry Pi. You should get an email after a couple of minutes.

