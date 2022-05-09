# Unremovable Files in UNIX

Let's say you have put your super malicious backdoor in a server, and you did not want anyone to remove this backdoor file. How to do this? Just make it immutable in file-system level :v

```bash
$ touch u-cant-rm-this

$ ls -l
total 0
-rw-r--r-- 1 root root 0 Mar 14 09:39 u-cant-rm-this

$ chattr +i u-cant-rm-this

$ ls -l
total 0
-rw-r--r-- 1 root root 0 Mar 14 09:39 u-cant-rm-this

$ rm u-cant-rm-this
rm: cannot remove 'u-cant-rm-this': Operation not permitted
```

Boom!

