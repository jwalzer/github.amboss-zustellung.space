---
description: Bericht aus dem Leben
lastmod: Tue Oct  7 16:31:46 2008
links: {}
stylesheets: []
tags:
- switching
- esx
- vmware
- trunk
- dhcp
- language/de
title: komplexe Systeme - Netzwerktroubles
---



Komplexe Systeme haben das Problem das sie komplex sind. Im Ernst!

Die letzte Woche war ich dabei, ein Kundenprojekt gerade zu ziehen, welches sich bereits über mehrere Monate gezogen hat.

Eigentlich war alles ganz einfach:

Der Kunde hatte ein bestehendes Netz und wollte mit der Zeit gehen und bei sich eine Logische Trennung zwischen verschiedenen Abteilungen und Netzen einführen. Seine bestehenden Server wollte er in eine neue VMware-ESX-Cluster-Umgebung umgezogen haben. "Wenn schon neu, dann richtig Flexibel" dachte sich der Kunde und entschied sich, seinen Clients anhand der MAC-Adressen in die passenden Netze zu verschieben. Statische Portkonfiguration kam bei der vorhandenen Hausverkabelung (und deren Dokumentation) nicht in Frage.

Dummerweise lief die Projektsteuerung suboptimal, was dazu führte, daß das Aufsetzen der ESX-Cluster und die Virtualisierung von einem VMWare-Spezialisten passieren sollte, bevor ein Netzwerker das Netzwerk am laufen hat. 
Außerdem wird natürlich alles schön sauber voneinander abgegrenzt, so daß die VM-"Spezies" keine Switche konfigurieren, und die Netzwerker keine VMs anfassen ... **fast**!

Also finde ich vor Ort ein Netzwerksetup vor, welches daraus besteht, daß der VM-Kollege einfach mal alle Switche, die er gefunden hat, zusammen gesteckt hat.

     "Dadurch konnte ich schonmal anfangen zu arbeiten, und hatte Netzwerk zum Einrichten der VMs"

Lustigerweise waren da nun zwei ESX-Server, jeweils mit 6GBit-Ports. Der Plan wahr wohl, je 1 Managment-Port (für die ESX) und 2 Datenports (für die VM-Kommunikation) pro Switch zu haben.
Dummerweise fliegt einem dann einiges mit der Redundanz um die Ohren, wenn man zwei Switche einfach nur verbindet, und dann das "NIC-Teaming"(In etwa wie LinkAggregation, nur ohne Negotiation) dafür benutzt, den Traffic wie beschrieben aus den Karten rauszupusten.
Nein, von Spanning Tree hatte der Kollege scheinbar noch nix gehört. von VLAN-Tagging erst recht nicht.
Und, ob es eine gute Idee ist einen Portchannel zu zwei verschiedenen Switchen aufzubauen, die dann wieder mit STP irgendwelche Loops erkennen(, oder auch nicht), darüber wurde garantiert kein Gedanke verschwendet.

Nungut, die gröbsten Probleme des Setup konnten dadurch behoben werden, daß wir sowieso auf Stacking zwischen den Switchen umgestellt haben. Dadurch konnte man dann zumindest die letzteren Themen sauber regeln. 
Blieb immernoch das Thema, daß der Kunde nun auch die Server per MAC-Addresse in ein eigenes VLAN verschieben wollte. Auf dem Switch!

Lustigerweise bieten die Switche eines bestimmten Herstellers, um den es hier geht, direkt das nette Feature, einen Port nicht im Access-Mode oder im Trunk-Mode zu betreiben, sondern, man kann ihn auch im "General-Mode" betreiben, in welchem man dann noch sagen kann, daß

  - alle VLANs auf diesem Port liegen,
  - diese VLANs untagged auf dem Port liegen
  - Ingress-Pakete erstmal in ein "native" VLAN gehen
  - außer es gibt eine MAC-VLAN-Assoziation, dann landen die Pakete in dem passenden VLAN

Zum Glück, mag man denken, bekommen die VMs ja eine statische MAC, welche auf diese Art dann einfach eintragen lässt, BINGO. Man Muß der Lösung zu gute halten, daß sie tatsächlich die Anforderungen erfüllt hat.

Fast.


Spaßig wurde es nämlich Probleme auf der Clientseite zu troubleshooten. Da hat nämlich scheinbar die VLAN-Zuordnung nicht funktioniert. Mal bekamen die Clients die richtige, und mal die falsche IP-Adresse vom DHCP. Und manchmal auch gar keine. Stundenlang wurde der DHCP-Relay auf den Switchen konfiguriert und getestet. Auf dem DHCP-Server (Windows-PDC) durften wir kein Wireshark installieren, sonst wären wir evtl. schneller voran gekommen. Ein DHCP-Server unter Linux auf dem Laptop, der an den Clientswitchen angeschlossen war, funktionierte über den DHCP-Relay wunderbar.
Also wurde der W2K-Server verdächtigt und ein W2K3-Server in einer anderen VM wurde schnell zum DHCP-Server gemacht ... Voila ... Das Problem sah' anders aus.

Jetzt bekamen die Clients dauerhaft eine falsche IP. Komischerweise mit kompletten DISCOVER/OFFER/REQUEST/ACK-Handshake. 

Es hatte relativ lange gedauert, bis uns ein-/auffiel, daß das Problem nicht auf Client- sondern auf der Serverseite lag. Auf dem zweiten Server konnten wir jedenfalls Wireshark installieren, und ziemlich schnell war das Problem klar. Dort kamen nämlich sowohl die Anfragen aus dem originalen VLAN als auch die Anfragen vom DHCPRelay in das separaten Servernetz an. Wie bitte? Klar!

Wie bereits erwähnt, liegen auf allen Serverports alle VLANs untagged an.... was das heißt, erstmal, ALLE VLANs kommen an dem ESX an, ohne eine Markierung welches VLAN das mal war. und die ESX gibt das natürlich auch an alle VMs weiter.

Irgendwann konnte ich mich mit diesen Beweisen soweit durchsetzen, daß mich der Kunde an seinen "Virtual Infrastructure Client" gelassen hat. Dort war es doch tatsächlich möglich innerhal des ESX-Servers einen Virtuellen Switch zu bauen, mit mehreren getaggten VLANs, die dann auch über die physischen Karten nach draußen zum Switch gereicht werden. Wenn man dann die Ports auf dem Switch einfach zum Trunk macht, dann spielt das alles ganz plötzlich ohne Probleme miteinander.

Traurig bleibt nur die Erkenntnis, daß dem VMware-"Spezialisten" die Begriffe VLAN, Tagging, Trunk oder Spanning-Tree nichts sagen.


