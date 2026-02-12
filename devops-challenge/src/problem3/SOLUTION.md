Provide your solution here:

First thing I do: df -h
I want to know Is / full? Or /var? Or /log?
then check 
sudo du -sh /* | sort -h
sudo du -sh /var/* | sort -h 
-> This quickly tells me which folder is eating the disk.

in production, this is almost always the reason.
NGINX handles traffic → traffic increases → logs grow → nobody rotates them → disk dies.
What I Usually Find
ls -lh /var/log/nginx/
can see 
access.log   28G bla bla 
error.log     6G bla 
Yep. That’s the problem.

- Emergency Fix 
sudo truncate -s 0 /var/log/nginx/access.log
sudo truncate -s 0 /var/log/nginx/error.log
or 
sudo rm /var/log/nginx/*.log
sudo systemctl reload nginx
Within seconds, disk drops from 99% → maybe 40%.

- Sometimes NGINX is fine, but system logs went wild.
running 
journalctl --disk-usage
If I see:
Archived and active journals take up 18G → problem found.
Why This Happens:
Errors looping/High traffic/No size limit -> fix by sudo journalctl --vacuum-size=500M

Stop the Bleeding -> Find the Root Cause -> Make Sure It Never Happens Again 