---
lastmod: Sun Apr 11 08:22:44 2010
links: {}
stylesheets: []
tags:
- debian
- linux
- initramfs
- shellscript
- rant
- blog
- WTF
- tech
- shell
- trouble
title: Fun with initramdisk - NOT!
---


#The Phenomenon

I just inherited the care for a system. The system of [SWSBO](http://acronyms.thefreedictionary.com/SWSBO) was setup by a good friend, a so called **Debian-Developer**, who knows what he does and has to do. Because of some ugly hardware issues he supplied the system with a custom build kernel because a year ago or so, some drivers for the suspicious hardware was not in mainline. The system is setup with LVM on the lower layer and having the single LVs encrypted with luks.  I normally setup my systems the other way around(puting the encryption below the LVM) but this gives equivalent security and should work equally well, if not even better.

Nevertheless the system looked quite goot in shape but meanwhile needed some updating.

So I did.

Imagine, your system refuses to build any new initramdisk. "**WTF**" I thought.

Yes, update-initramfs  and mkinitramfs barfs out.

          mkinitramfs: for root /dev/mapper/marte-c_root_crypt missing dm- /sys/block/ entry
          mkinitramfs: workaround is MODULES=most
          mkinitramfs: Error please report the bug

Wow! ... thats a thing. Of course what I can see is, that it seems to have the opinion the root-device is depending on "dm-" -- obviously missing a number at the end. But where does that come from?
No information is available on how to debug this. How does mkinitramfs come to this value at all?
Neither /proc/mounts nor /etc/{crypt,m,fs}tab had a rouge entry that could lead to this.


#is the source with me

So what's happening here? Diving into updateinitramfs doesn't bring any clue, because it's simply calling mkintramfs with some parameters.

Next try:

          sh -x mkinitramfs .......

shows us the flow of the execution and where it barfs out. Not in mkinitramfs. 

You can find it in /usr/share/initramfs-tools/hook-funktions -- Who would have thougt that ...

So I found the source, and, couldn't really believe. It's absolutely doing what the author intended. There seems no bug, or strange case here.

The Author didn't think of encrypting an logical volume. I was even told in IRC that this code can't deal with more than 2 Layers  of devicemapper-dependencies. I don't quite understand, where the difference in Nr-of-Layers is in comparision to having the crypt below the LVM, but I didn't care any longer at that moment for a discussion. I wanted a solution.

#Finding the Solution

Now, even the error mentions the solution: "MODULES=most" -- This should disable the "*pseudointelligent*" algorithm thats not able to find the correct rootdevice, but instead put nearly all sensible modules into the ramdisk.

"Strange: Didn't I already configure that?"

          ~# grep MODULES /etc/initramfs-tools/initramfs.conf 
          # MODULES: [ most | netboot | dep | list ]
          MODULES=most

Indeed! ... so what's the matter here?

A slight modification made my head slap for another hour against the wall. 

          ~# grep **-r** MODULES /etc/initramfs-tools/*
          /etc/initramfs-tools/initramfs.com:# MODULES: [ most | netboot | dep | list ]
          /etc/initramfs-tools/initramfs.com:MODULES=most
          /etc/initramfs-tools/conf.d/driver-policy:MODULES=dep

So we can see, that some obscure script, put this file there, and happily overwriting my manual changes in the default configuration-location. 

#The Solution

The problem was solved with:

          rm /etc/initramfs-tools/conf.d/driver-policy

Of course, I verified, that this file only consisted of this very line and some comments "Attention, this will override global config!" -- No! Indeed! Who would have thought!


#Resume

I still haven´t quite understood how the code in mkinitramfs should work, but somehow it seems a bit flawed. A can imagine a lot other setups that would lead to to stranger problems.
I will see, if I can manage to write a bugreport, and will update here when I have a BTS#. Meanwhile I'm wondering, how fragile this whole kernel/Boot-Infrastructure is, and why I am the one who happens to stumble over these uggly problems.



