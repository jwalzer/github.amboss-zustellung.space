---
lastmod: Thu Jan 15 21:58:12 2009
links: {}
stylesheets: []
tags:
- rant
- blog
- samsung
- harddrive
- storage
- qnap
- smart
title: Samsung Spinpoint und SMART
---
[[!meta title="Samsung Spinpoint und SMART" ]]

# Samsung Spinpoint mit 1TB

"Naja, man gönnt sich ja sonst nix. Und Billig ist es heutzutage auch." dachte ich noch letztes Jahr, als ich beschloss, mir ein QNAP TS-409 zuzulegen.

Das ist so ein schönes, nettes, schniekes Netzwerk-Speicher-Gerät, welches Out-Of-Box einfach mal mit einer Menge Services funktioniert (OK, für NFS brauchte es mangels der Pro-Version einen kleinen Hack)

Das Gerät unterstützt bis zu vier SATA-Festplatten und kann diese als Raid 0,1,5,6 oder JBOD betreiben. Ursprünglich hatte ich mit Raid6 geliebäugelt, aber damit ist der kleine Prozessor ein wenig überfordert und liefert mir schreibraten unterhalb der 1MB/s Grenze ... sorry, das kann man sich sparen.

Zähneknirschend nutze ich jetzt Raid5 mit einer (fehlenden) Spare.
Warum fehlend? ... genau deshalb schreibe ich grad 'den Artikel.

## Vier Festplatten sollen es sein

"... Jede 1TB groß", war die Devise. Also doch mal durch die einschlägigen Onlineversender durchgeschaut, Kundenbewertungen gelesen, das Offizielle von QNAP nach der Kompatibilitätsliste abgeklappert.

Ergebnis: Western Digital und Samsung haben beide sowohl spitzen als auch mieseste Bewertungen bekommen. Ich weiß selbst, daß jemand enttäuschtes lauter und böser seine Meinung kundtuen wird, als jemand zufriedenes ... und Ausreißer passieren ... also unentschieden.

Um Ausfallwahrscheinlichkeiten zu verringern, hab' ich dann 2 mal die WDC WD10EACS-65D6B0 01.0 und zweimal die SAMSUNG HD103UJ 1AA0 bestellt. 

# Und dann fing die Odyssee an ...

Die Samsungs melden sich dem Kernel soweit ich das sehe identisch. Allerdings liefert die eine Platte SMART-Daten, und die andere nicht.

"Naja, umtausch halt ... "
Tja, mit dem Gedanken, spiele ich das Spiel jetzt schon seit Ende Oktober. 

  1. Im Web-Interface des Versenders Retour ankündigen
  1. Austausch verlangen
  - (optional) telefonisch dort das Problem vortragen
  - Kleines Anschreiben mit Fehlerdiagnose ausdrucken
  - Festplatte ausbauen
  - Festplatte einpacken
  - Anschreiben dazu
  - Retourschein aufbappen
  - Zur Post bringen
  - warten
  - 1-2 Wochen Warten
  - warten
  - Päckchen in Empfang nehmen
  - Päckchen auspacken
  - Verpackung nicht zu weit wegräumen
  - Platte einbauen
  - SMART überprüfen
  - GOTO 1.

# Es Nervt!

Dank der Hotswap-Rahmen und SATA geht so ein Ein/Ausbau heutzutage ja erfreulich zügig. Aber **Es Nervt!**

Ich bin jetzt wahrscheinlich schon bei der 5. Platte, die nicht in der Lage ist korrekt SMART zu sprechen.
Aber warum? 

Die erste Samsung Platte, die identisch sein sollte, tut dies ohne Probleme. Und die Slots des Gehäuse wurden auch schon getauscht (bzw. die Platten darin). Weder liefern die getauschten Platten in einem anderen Slot korrekte Daten, noch ist es so, daß eine der funktionierenden Platten in dem Slot keine Daten liefert.

Meine aktuelle These bezieht sich auf die Firmware-Version der Platten. Die Frage ist nur, ob die funktionierende Platte eine neuere oder eine ältere Firmware hat. 

Die Firmware des Devices ist soweit ich es vertreten kann auf einem aktuellen Stand (nein, ich mag nicht jede neueste Beta einspielen, da ich inzwischen doch schon einiges wieder darauf habe).

# Dear Lazyweb

Hat irgendwer schonmal von Problemen bzgl. SMART und Samsung Spinpoints gehört oder kennt abhilfe?

Ich werde das Spiel mit dem Versender noch eine Weile spielen ... 


