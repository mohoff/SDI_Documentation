
***
DNS
***

DNS intro mit HdM addresses
###########################

nslookup hdm-stuttgart.de geht nicht
nslookup www.hdm-stuttgart.de geht
nslookup mi.hdm-stuttgart.de geht

Domainnamen -> IP-Adressen
Rechnernamen (in der Domaene) <-> IP-Adressen
mx-records
Nameservce
(logisch) <-> (physisch)

wenn der rechner in der gleichen Domaene ist, kann auch nslookup <rechnername> eingegeben werden.
Sonst mit nslookup <rechnernetze>.<domain> erreichbar

Warum braucht man DNS?
######################
- IP Adressen aendern sich.
- IP Adressen kann man sich schlecht merken

bla
###

nslookup eingeben fuer nslookup-CLI:
> set type=mx
> mi.hdm-stuttgart.de
(mehrere maschinen mx1,mx2,..,mx5.hdm-stuttgart.de)
warum mehrere? Lastverteilung, in gewissen Graden Ausfallsicherheit
DNS kann mit lastverteilungs-prozenten konfiguriert werden, damit grosse mailserver z.b. mehr
traffic bekommen wie kleine mailserver.

bei www.google.de kommt nur ein Eintrag zurueck. bei google.de kommen mehrere zurueck (PTR-verweis?)

>set type=ns
>mi.hdm-stuttgart.de
(hat 3 eintrage, wegen ausfallsicherheit)
>hdm-stuttgart.de
(hat 5 eintrage, 3 davon intern, 2 von belwue [forschungsnetz]. belwue verlangt dass 2 DNS ausserhalb
der einrichtung ist)


links im freedocs-script  (erster ist basic, zweiter komplex, dritter fuer die heutige vorlesung gut, ganz detailliert im vierten link (isc.org))

Es gibt ein DNS-secure, da es viele DNS-angriffsszenarien gibt (spoofing vor allem...)

Man braucht 2 Zonen um einfachen DNS-server einzurichten.
1.) Forward-Zone: Rechnername -> IP-Adresse
2.) Reverse-Zone: IP-Adresse -> Rechername

DNS-Forwarding
##############
Wenn Leaf-DNS etwas nicht aufloesen kann, geht die Anfrage weiter an uebergeordnetes DNS, das evtl. mehr weiss.

Relevant: Attribute DNSSEC



>nslookup
>server sdi1a.mi.hdm-stuttgart.de
(jetzt gehen die anfragen an diesen DNS-server und fraegen dessen eintraege ab)


Logs sind default-maessig in ``/var/log``. Nicht DNS-spezifisch, sondern allgemeiner log-Ordner fuer viele Dienste. Uns interessiert ``syslog`` (da steht u.a. DNS-log drin, auch LDAP (slapd) drin). Wenn logfiles zu gross werden, werden sie separat unter neuem Namen abgespeichert. Es gibt einen Service, der das uebernimmt. Mit ``tail`` laesst sich aber das "Ende einer Datei" ausgeben. ``tail syslog`` in ``/var/log``. ``tail -f syslog`` macht das live, sprich pollt die Datei regelmaessig und gibt immer die neusten Zeilen aus. Bei ``service bind9 restart`` wird DNS-Service neu gestartet und ``tail -f syslog`` spuckt neue Zeilen aus. Trotzdem koennen noch zu viele Eintraege kommen. Mit Filtern wie ``tail -f syslog | grep named | grep loaded`` laesst sich die Ausgabe weiter einschraenken.


Datei ``/etc/resolv.conf`` ist fuer ... gut.

Einzelne Hosts koennen in ``etc/hosts`` eingetragen werden. Da steht meistens local loopback drin.

Hostname eines Rechners mit ``hostname`` zu bestimmen.

