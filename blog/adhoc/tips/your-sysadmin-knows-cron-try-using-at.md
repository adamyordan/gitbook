# Your sysadmin knows CRON? Try using AT

You want to make your backdoor persistence? One of the easiest thing is to just add it to cron. However, everybody knows cron! Of course it will be the first place to check by your sysadmin, and your malicious backdoor will be gone as easily as you put it on the cron.\
\
Try using **at**. Of course if someone knows it, it is also very easy to detect and remove, but I'll bet there are far fewer people that knows that the command even exists.

```bash
$ echo "echo 1337 > /tmp/meh" > malicious.sh

$ at -f malicious.sh now + 1 minute
warning: commands will be executed using /bin/sh
Job 1 at Thu Mar 14 18:54:00 2019

$ atq
1 Thu Mar 14 18:54:00 2019 a user

# wait...

$ cat /tmp/meh
1337
```

But it only run once! What to do if you want to run this periodically?\
Just add at command in your scripts...

```bash
$ echo "at -f malicious.sh now + 1 minute" >> malicious.sh
$ at -f malicious.sh now + 1 minute
```
