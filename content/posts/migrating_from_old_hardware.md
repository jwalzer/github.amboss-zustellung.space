---
lastmod: Wed Jan  7 23:07:22 2009
links: {}
stylesheets: []
tags:
- rant
- blog
- memtest86
- old_hardware
title: migrating_from_old_hardware
---
[[!meta title="migrating_from_old_hardware" ]]


# dear lazyweb ...


* what does it mean, when memtest86+ suddenly stops in a test?
* what does it mean, if it is always the same test?
* what does it mean, when I even shuffle around the memorymodules and the slots, and ALWAYS memtest86+ stops when  doing Test#7 (Random number sequence )?

I mean, I've read a lot stuff now on the net (because memtest runs take a lot time) and found NOTHING on such an issue. 
Some single reports that state that mem86+ shouldn't crash but rather report an error when it finds a problem.

All the other tests were running fine without problems.

In the meanwhile I've disabled anything that could hide the effect, such as legacy on-board sound, usb, and even 
disabled L1 and L2 cache.

Et voila -- now I just selected Test#7 directly and it gives me nice red lines of wrong data ... yeah, so far I assume the first half MByte was okay, but so far I dont't believe there was anything correct at all in this test.

Should I conclude now, that there's a problem with the processor? with the mainboard?

## But Why ?

What the heck do I care at all for this "old" Athlon 1.3GHz ?

Now; This one is faster than the "old server". I need to shutdown [Zion](Zion). Zion was old and served me well the last  8 years. Its hardware was shiny new when I bought it 10 years ago. 
But today its old. Its loud. And it costs to much power (I got an server PSU for it in these days). This all wouldn't have stopped me but it was no longer reliable. Zion still holds my personal uptime record with ~620 days,  but these were the times when 2.4 kernelseries was current.
In the last year it happend nearly every fortnight that the system simply hung. The Problem was, that when I moved I left this machine still running at my parents home, because it was managing the network there and remotely for me and I had not yet found the time to restructure that. So I had no quick way to reboot that box when needed.

So in the last year I tried to move critival services to a hosted dedicated server, which leaves me with the question what to do with the datastorage. 
Zion has had a raid1 with 750GB capacity. so I decided to buy an QNAP TS409 with 4x1TB and copy the stuff over. A quick 
calculation gave that it wouldn't work out to copy that stuff on a weekend when we're visiting my parents. Zion was good enough the old days, but it could neigther handle ~2*10^9 Files (rsnapshot at work) via rsync, nor would it be able to fill the 100MBit card at all. Tests showed, that it could read from the filesystem with ~5MB/s when I would  just be using `tar | netcat` so you can do the maths yourself... 

So I took Zion and my former desktop (the Athlon) with me to merge them and to copy the data over.

This gave me some struggles. First I needed to buy an old used tft as I have only laptops in my home so I could at least see whats happening. 
Now I'm here, moved the old raid system over to the athlon and can't boot the old system, because the initrd doesn't find the ide devices. 

## "Use GRML" I hear you say...

of course ... of course I use grml ... at least I want to.

but this board doesn't boot from USB... So I had to find some CD-Rs ... I had to find a working CD-Rom-Drive ... and burn the  disc.

Just to find out, that it always crashes. ... So I'm now using the memtest86+ (v1.70) on the grml image to check the  memory and reproduced the effect that the random number sequence test#7 hangs the machine, huh?

So I debug the memory modules, the memory slots .. and now .. after changing the bios-settings to some **very**  conservative values, it seems to no longer crash, but instead gives me the whole memory module as faulty? ... BTW: After 3hours it already gave me the first 32MB as faulty ... still running

## And why ?

Because there is no other box here available to me that I could connect the four PATA-harddrives from zion to. In 
fact: There are only laptops and embedded boxes here around. And after having these two boxes here for a couple of weeks now, I know thats because of the sound too.

Grrr... I'm gooing to look for a cheap, old, used P4 somewhere around the corner here, that will be known to work.  Put maybe some RAM into it, and hook a grml to the usb and the drives to the pata ... this will be probably quicker 

... 



