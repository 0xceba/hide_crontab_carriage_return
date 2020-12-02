# Hide Crontab Contents with a Carriage Return

### What's happening?

The contents of a crontab file can be hidden by inserting a carriage return character and a fake message. When a defender lists the crontab with `crontab -l`, the carriage return character will be interpreted so the defender will see the message instead of the scheduled task. This is a simple trick which can be used to hide maintain persistence.

### Where did this come from?

This maneuver comes straight from [@tylerni7](https://twitter.com/tylerni7)'s mouth in [epsiode 43](https://darknetdiaries.com/episode/43/) of [Darknet Diaries](https://darknetdiaries.com/). In this episode, Tyler is describing PPP's participation in one of Defcon's annual attack-and-defend CTF events. During this event, Tyler's team knew that an adversary had gained access to one of their machines and was exfiltrating the flag. They thought the adversary was accomplishing this through a cron job but when they listed the jobs with `crontab -l` they saw the message `no crontab for [user]`. After seeing this message they assumed the data was being exfiltrated a different way and moved on.

Later they found out that there _was_ a cron job but that the adversary had included a carriage return character with a phony 'no crontab' message in the crontab. This caused the `crontab -l` command to display the adversary's message instead of the cron job. 

### Why does this work?

Some Linux binaries (like `cat`, `head`, `tail`, and `nl`) interpret control characters and others (`less`,`more`) display or ignore them:
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

`crontab -l` treats the control characters the same way that `cat` and `head` do: `crontab -l` interprets the carriage return as an instruction and moves the cursor to the front of the line.

### How do we execute this?

**Manual execution:**
1. Editing the current user's crontab: `crontab -e`
1. Deleting the preamble:
	* nano:
		1. `Ctrl`+`Shift`+`6` to select the first line of the preamble.
		1. Move cursor with down arrow key until the final line.
		1. `Ctrl`+`K` to cut/delete the selected text.
	* vi(m):
		1. Issue command `23dd` from the first line to delete the first 23 lines (length of the preamble).
1. Inserting the scheduled task with a trailing semicolon: `* * * * * touch /tmp/1;`
1. Inserting the carriage return:
	* nano: `Alt`+`V` then `Ctrl`+`M`.
	* vi(m): `Ctrl`+`V` then `Ctrl`+`M`.
1. Writing the display message that says there are is crontab: `no crontab for [user]`
1. Adding sufficient whitespaces after the message to completely hide the crontab contents. The formula for the minimum number of whitespace is: length of cronjob - 14 - length of user name.

When finished the crontab file will contain the following:
```
* * * * * touch /tmp/1;^Mno crontab for root                             
```

**Bash one-liner:**
```
cc="* * * * * touch /tmp/1" ; ct=$(printf "$cc;\rno crontab for $USER%*s" $(expr length "$cc" - 14 - $(expr length $USER))) ; echo "$ct" | crontab -
```

What the defender sees when they list the user's crontab using `crontab -l`:
```
root@kali:# crontab -l
no crontab for root                             
```