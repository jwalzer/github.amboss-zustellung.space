---
lastmod: Mon Feb 15 11:16:27 2010
links: {}
stylesheets: []
tags:
- rant
- blog
- linux
- windows
- skype
- IM
- instant
- messaging
- pidgin
- xmpp
- WTF
title: Doing Videomessaging between Windows and Linux
---


#Dear Lazyweb ...

I don't like skype. It's closed source. The protocol has been barely reverse engineered.

So I'd rather go with xmpp(jabber). Jabber is an quite cool protocol and there  are enough clients out there, that are able to speak xmpp.

But: there has been at least 5 years in between the first Instant Messengers that could do Voice and even Videochats.

Why is it so complicated to find an xmpp-Client for Windows that's able to do that?  

I mean, come on: Windows is the base of the "just want it to work"-userbase. Skype simply does. Even MSN simply works and is preinstalled.

How can I explain a "standarduser" we have to sort out and test different xmpp-clients to have webcam and phone?

I'm using myself pidgin on linux. It has support for Voice and Audio ... great. Even works really well between two linux machines. 
But pidgin on windows? No support. Yes, there may be some reasons. Pidgin has build in Multimedia-Stuff via *gstreamer* subsystems. These are not available on windows ...

OK, look for an other xmpp-client, thats available for windows. Even tried the complete [Wikipedia List of XMPP-Clients](http://en.wikipedia.org/wiki/List_of_XMPP_client_software). 

- *Pidgin* has all the plugins I'm using but can't support Video/Audio on Windows because of no gstreamer support
- *Miranda* had a chance, but Videosupport was only proprietary and from 2006(?) ... 
- *Psi* has only audio support, but couldn't negotiate with linux (but it tried, at least) (BTW: This support is by an gstreamer.dll as an MinGW-compile. WTF: Pidgin, do you hear me?)
- *Jabbear*(notice the "a"!) downloads an version, installs. when startet it tells me, that this beta is expired, and there is a new version available. Donwload it, damned! "Uhh,ohh .. can't download ... ". WTF?!?!
- *SIP-Communicator* has support for xmpp and for voice/video ... but the latter only for SIP! *WTF!!!*

How far should one go from here?

My requirements are quite simple:
Make Voice and Video chats between Linux and Windows users as simple as *skype* but use opensource, preferably XMPP as protocol. Support for OTR  has a really high priority. 


Dear lazyweb ....

  


[[!inline pages="./discussions/*" rootpage="./discussion" postformtext="Comment" ]]

