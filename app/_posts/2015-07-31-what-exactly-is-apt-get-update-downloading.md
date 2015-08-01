---
layout: post
title:  "What, exactly, is apt-get update downloading?"
date:   2015-07-31 22:30:00
tags:   apt
---
So I'm struggling with my ISP at the moment (whatever you do, do _not_ sign up with [izzi](http://izzi.mx) because they absolutely don't know what they're doing), and I've noticed that for a few days now, my apt updates have been taking an extremely long amount of time to finish.

![aptitude trying to update](/images/Screenshot 2015-07-31 23.08.32.png)

There's clearly something going on there, but neither apt-get nor aptitude give you much info to work with as things happen. You _can_, however, find out what files it downloads in order to determine whether updates are available or not.

Here's how you obtain a list of all those files:

~~~sh
$ apt-get update --print-uris   # No root required for this one
~~~

This gives you a list that looks like this:

~~~
'http://archive.ubuntu.com/ubuntu/dists/vivid/main/source/Sources.bz2' archive.ubuntu.com_ubuntu_dists_vivid_main_source_Sources 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/source/Sources.bz2' archive.ubuntu.com_ubuntu_dists_vivid_restricted_source_Sources 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/universe/source/Sources.bz2' archive.ubuntu.com_ubuntu_dists_vivid_universe_source_Sources 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/multiverse/source/Sources.bz2' archive.ubuntu.com_ubuntu_dists_vivid_multiverse_source_Sources 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/main/binary-amd64/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_main_binary-amd64_Packages 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/binary-amd64/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_restricted_binary-amd64_Packages 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/universe/binary-amd64/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_universe_binary-amd64_Packages 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/multiverse/binary-amd64/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_multiverse_binary-amd64_Packages 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/main/binary-i386/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_main_binary-i386_Packages 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/binary-i386/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_restricted_binary-i386_Packages 0 :
'http://archive.ubuntu.com/ubuntu/dists/vivid/universe/binary-i386/Packages.bz2' archive.ubuntu.com_ubuntu_dists_vivid_universe_binary-i386_Packages 0 :
...
~~~

As you can see, the first part is the URL of the file to download.

To turn this mess into something I can use, I did this:

~~~sh
$ apt-get update --print-uris | gawk '{print $1}' |\
    grep 'bz2' | sed "s/'//g" > apt-list.txt
~~~

Which gives me just the URLs, one per line:

~~~sh
$ cat apt-list.txt
http://archive.ubuntu.com/ubuntu/dists/vivid/main/source/Sources.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/source/Sources.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/universe/source/Sources.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/multiverse/source/Sources.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/main/binary-amd64/Packages.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/binary-amd64/Packages.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/universe/binary-amd64/Packages.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/multiverse/binary-amd64/Packages.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/main/binary-i386/Packages.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/restricted/binary-i386/Packages.bz2
http://archive.ubuntu.com/ubuntu/dists/vivid/universe/binary-i386/Packages.bz2
...
~~~

I can now do whatever I want with this. For example, I used wget to download them all, and took note when I noticed a particular file having issues trying to download:

~~~sh
$ wget -i apt-list.txt --progress=dot -O /dev/null
~~~

And then, with the problematic URL, do the following to get the actual time spent:

~~~sh
$ /usr/bin/time wget http://dl.google.com/linux/chrome/deb/dists/stable/main/binary-amd64/Packages.bz2 -O /dev/null
--2015-07-31 22:51:27--  http://dl.google.com/linux/chrome/deb/dists/stable/main/binary-amd64/Packages.bz2
Resolving dl.google.com (dl.google.com)... 64.233.169.93, 64.233.169.190, 64.233.169.136, ...
Connecting to dl.google.com (dl.google.com)|64.233.169.93|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1167 (1.1K) [application/octet-stream]
Saving to: ‘/dev/null’

/dev/null                 100%[=======================================>]   1.14K  --.-KB/s   in 0s

2015-07-31 22:51:27 (94.0 MB/s) - ‘/dev/null’ saved [1167/1167]

0.00user 0.00system 3:20.19elapsed 0%CPU (0avgtext+0avgdata 4364maxresident)k
0inputs+0outputs (0major+252minor)pagefaults 0swaps
~~~
