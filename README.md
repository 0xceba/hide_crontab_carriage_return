# Hide Crontab Cron Job with a Carriage Return

### What is the tactic?

A cron job in a crontab file can be hidden by inserting a carriage return character and a fake message. When a user's crontab is listed with `crontab -l`, the carriage return character will be interpreted so the fake message will be displayed instead of the cron job. This is a simple trick which can be used to maintain persistence.

### Where did this come from?

This maneuver comes straight from [@tylerni7](https://twitter.com/tylerni7)'s mouth in [epsiode 43](https://darknetdiaries.com/episode/43/) of [Darknet Diaries](https://darknetdiaries.com/). In this episode, Tyler is describing PPP's participation in one of Defcon's annual attack-and-defend CTF events. During this event, Tyler's team knew that an adversary had gained access to one of their machines and was exfiltrating the flag. They thought the adversary was accomplishing this through a cron job but when they listed the jobs with `crontab -l` they saw the message `no crontab for [user]`. After seeing this message they assumed the data was being exfiltrated a different way and moved on.

Later they found out that there _was_ a cron job but that the adversary had included a carriage return character with a phony 'no crontab' message in the crontab. This caused the `crontab -l` command to display the adversary's message instead of the cron job. 

### Why does this work?

Some Linux binaries (like `cat`, `head`, `tail`, and `nl`) interpret control characters and others (`less` and `more`) display or ignore them:
```
root@kali:# echo -e '1\r2' > cr
root@kali:# cat cr
2
root@kali:# head cr
2
root@kali:# tail cr
2
root@kali:# nl cr
2    1  1
root@kali:# less cr
1^M2
root@kali:# more cr
1
```

`crontab -l` treats the control characters the same way that `cat` and `head` do: `crontab -l` interprets the carriage return as an instruction so `crontab -l` moves to the beginning of the line before continuing to write to stdout.

### How is this tactic executed?

**Manual execution:**
1. Edit the current user's crontab: `crontab -e`.
1. Delete the preamble.
1. Insert the scheduled task with a trailing semicolon: `* * * * * touch /tmp/1;`.
1. Insert the carriage return:
	* nano: `Alt`+`V` then `Ctrl`+`M`.
	* vi/vim: `Ctrl`+`V` then `Ctrl`+`M`.
1. Write the display message that says there is no crontab: `no crontab for [user]`.
1. Add sufficient whitespace after the message to completely hide the cron job. The minimum number of whitespace is: length of cron job - 14 - length of username.

The crontab file will look like the following when finished:
```
* * * * * touch /tmp/1;^Mno crontab for root    
```

**Bash one-liner execution:**
Or use the following one-liner:
```
cc="* * * * * touch /tmp/1" ; ct=$(printf "$cc;\rno crontab for $USER%*s" $(expr length "$cc" - 14 - $(expr length $USER))) ; echo "$ct" | crontab -
```

### What does the defender see?
```
root@kali:# crontab -l
no crontab for root    

root@kali:# cat /var/spool/cron/crontabs/root 
# DO NOT EDIT THIS FILE - edit the master and reinstall.
# (- installed on [redacted])
# (Cron version -- $Id: crontab.c,v 2.13 1994/01/17 03:20:37 vixie Exp $)
no crontab for root    
```