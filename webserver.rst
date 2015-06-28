
****************
Apache Webserver
****************

Webserver Einführung
####################
Die Aufgabe eines Webservers ist es, statische Dateien zur Darstellung von Webseiten auf Anfrage auszuliefern. Bei den Dateien handelt es sich vorrangig um HTML-Dokumente, Stylesheets und Bild-Dateien. Jede dieser Dateien wird über eine separate Anfrage angefordert. Üblicherweise verwendet man für diesen Vorgang das HTTP-, bzw. HTTPS-Protokoll. Der Ablauf von der Anfrage bis zur Darstellung einer Webseite kann grob in folgenden Schritten zusammengefasst werden:

* Der Benutzer gibt in die Adresszeile des Browsers die URL der gewünschten Seite ein
* Falls die URL einen Domainname enthaelt, wird dieser zunaechst per DNS auf eine IP-Adresse aufgeloest. An diese IP-Adresse wird eine HTTP-Anfrage auf Port 80 geschickt.
* Der Server empfängt die Anfrage und:

  * ent- und verschlüsselt Anfrage und Antwort, falls TLS eingesetzt wird 
  * überprüft ggf., ob der der Nutzer zum Empfang der Ressource berechtigt ist (Authentifizierung/Authorisierung)
  * generiert ggf. dynamische Teile der Website
  * protokolliert ggf. die Anfrage
  * cached ggf. die Anfrage für zukünftige Anfragen der gleichen Ressource
* Er antwortet mit der angeforderten Seite, bzw. einer Fehlerseite, falls die Ressource nicht gefunden wurde bzw. die Authentifizierung fehlgeschlagen ist.
* Der Browser des Clients interpretiert die vom Webserver erhaltenen Daten, fuehrt die Skripte aus, rendert die Seite und zeigt sie dem Benutzer an.

Virtual Hosts
#############
Falls auf einem Webserver mehrere Websites gehostet werden sollen, benötigt man Virtual Hosting. Es gibt zwei Techniken um dies zu erreichen: IP- und namensbasiertes virtuelles Hosting.

IP-basiertes Virtual Hosting
****************************

Beim IP-basierten Virtual Hosting wird jedem virtuellen Host eine eigene IP-Addresse zugewiesen. Der Webserver hat dafür mehrere Netzwerkschnittstellen (physikalisch oder virtuell) mit verschiedenen IP-Addressen.

Im Endeffekt kann der Server anhand der IP- Adresse, auf der eine Anfrage angekommen ist, entscheiden, welcher virtuelle Host angesprochen wurde.

::

    <VirtualHost 192.168.0.1:80>
      ...
    </VirtualHost>
    <VirtualHost 192.168.0.2:80>
      ...
    </VirtualHost>

Namensbasiertes Virtual Hosting
*******************************

Namensbasierte virtuelle Hosts verwenden mehrere Hostnames mit derselben IP. Dies erlaubt dem Server, mehrere Webseiten auf derselben IP zu betreiben. Der Server liest diese Information aus und reagiert entsprechend. Mit dieser Technik koennen mehrere Hosts ueber eine IP-Adresse betrieben werden. Der Vorteil ist, dass trotz IP-Adressenknappheit mehrere Hosts betrieben werden koennen.

Voraussetzungen:
* Browser gibt den gewuenschten Hostname im HTTP-Header seiner Anfrage mit an.
* DNS-Eintraege muessen so gewaehlt sein, dass sie alle in Frage kommenden Domainnamen auf die gleiche korrekte IP-Adresse uebersetzen.

::

    NameVirtualHost *:80    // fuer alle IP-Adressen wird auf Port 80 namensbasiertes virtuelles Hosting verwendet. Wenn nur "*" steht, gilt das sowohl fuer HTTP und HTTPS.

    <VirtualHost *:80>
      ServerName www.domain.tld
      ServerAlias domain.tld *.domain.tld
      DocumentRoot /www/domain
    </VirtualHost>

    <VirtualHost *:80>
      ServerName www.otherdomain.tld
      DocumentRoot /www/otherdomain
    </VirtualHost>


Apache Funktionsweise
#####################

Apache wird standardmaessig in das Verzeichnis ``/etc/apache2/`` installiert. Dieses Root-Verzeichnis enthaelt zu Beginn folgende Dateien und Unterordner:

::

    apache2.conf
    conf-available/
    conf-enabled/
    envvars
    magic
    mods-available/
    mods-enabled/
    ports.conf
    sites-available/
    sites-enabled/

Das Standardverzeichnis fuer Webinhalte ist unter ``/var/www/html/``.

Im Folgenden wird auf die vier Dateien ``apache2.conf``, ``envvars``, ``magic`` und ``ports.conf`` eingegangen, sowie auf die sechs Unterordner, die *verfuegbare* (``available``) und *aktivierte* (``enabled``) Konfigurationen, Mods und Seiten enthalten.

``apache2.conf``
****************
``apache2.conf`` ist eine der beiden ``.conf``-Dateien, mit denen sich der Apache-Webserver konfigurieren laesst (frueher: ``httpd.conf``). Als Hauptkonfigurationsdatei fuegt die Teile anderer Konfigurationsdateien (``mods``, ``confs``, ``sites`` und ``ports.conf``) zusammen, wenn der Server gestartet wird. Beispielhaft sind hier ein paar wichtige Einstellungen aufgefuehrt:

* ``ServerRoot``: Root-Verzeichnis, unter dem die Konfigurationen, Error- und Logdateien des Servers gehalten werden. Bsp.: ``/etc/apache2``.
* ``Timeout``: Anzahl Sekunden, die gewartet wird, bevor ein Timeout-Signal gesendet wird. Bsp.: ``300``.
* ``KeepAlive``: Ob permanente Verbindungen (mehrere Anfragen pro Verbindung) erlaubt sind oder nicht. Bsp.: ``On``.
* ``MaxKeepAliveRequests``: maximale Zahl an Anfragen, die pro persistenter Verbindung erlaubt sind. Bsp.: ``100``.
* ``HostnameLookups``: Ob nur die IP-Adresse oder auch der Hostname ueber einen versuchten DNS-Lookup in Logdateien gespeichert wird. Hat zur Folge, dass pro eingehender Verbindung mindestens 1 Lookup stattfindet. Bsp.: ``Off``.
* ``ErrorLog``: Pfad der Errorlog-Files. Dient als Fallback, wenn die virtuellen Hosts diesen Wert nicht setzen. Bsp.: ``${APACHE_LOG_DIR}/error.log``.
* ``Include`` bzw. ``IncludeOptional``: Andere Konfigurationsdateien werden eingebunden. Bsp.: ``ports.conf``.
* ``LogLevel``: Gibt die "Strenge" an, mit der Nachrichten gelogged werden sollen. Bsp.: ``warn``.
* globales, default Security-Model mittels ``Directory``-Direktiven:
  
  ::
  
      <Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
      </Directory>
      
      <Directory /usr/share>
        AllowOverride None
        Require all granted
      </Directory>
      
      <Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
      </Directory>

  Damit wird der Zugriff auf das Root-Filesystem explizit verboten (erste Direktive) und der Zugriff auf ``/usr/share/`` und ``/var/www/`` erlaubt (zweite und dritte Direktive). Host-spezifische (Directory-)Direktiven koennen in den entsprechenden VirtualHost-Direktiven in ``/etc/apache2/sites-available`` festgelegt werden.
* ``AccessFileName``: Der Name der Datei, die in jedem Ordner gesucht wird, um nach zusaetzlichen Konfigurations-Direktiven zu schauen.
* ``<FilesMatch "^\.ht">Require all denied</FilesMatch>``: Mit dieser Direktive koennen die Dateien ``.htaccess`` und ``.htpasswd`` nicht von Clients gelesen werden.
* ... und einige Umgebungsvariablen, z.B. ``${APACHE_PID_FILE}``, die aus der Datei ``envvars`` referenziert werden.


``ports.conf``
**************
``ports.conf`` wird immer von ``apache2.conf`` eingebunden. Es enthält Direktiven, die festlegen, auf welchen TCP-Ports Apache lauschen soll.

``envvars`` und ``magic``
*************************
In ``envvars`` werden, wie der Name schon erahnen laesst, Apache-Umgebungsvariablen gesetzt.

``magic`` enthaelt Regeln, um anhand der führenden Bytes einer Datei dem MIME-Typ zu erkennen.


``conf-available`` und ``conf-enabled``
***************************************
bla

``mods-available`` und ``mods-enabled``
***************************************
bla

``sites-available`` und ``sites-enabled``
*****************************************
bla

Apache Befehle
##############
* a2ensite, a2dissite
* a2enmod, a2dismod (?)
* a2enconf, a2disconf (?)
* apache2 -v (gibt versionsnummer und built-timestamp aus)
* apache2 -t (checked syntax von config-files)
* service apache restart,reload,start,stop,force-reload
https://wiki.ubuntuusers.de/apache#Apache-steuern

Apache Prozesse
###############
Wie in folgendem Auszug aus der Konsole zu sehen ist, existieren mehrere zu Apache zugeordnete Prozesse gleichzeitig wenn der Webserver gestartet ist. Grund hierfuer ist, dass bei Serverstart ein ``apache2``-Prozess vom User ``root`` gestartet wird, der die TCP-Ports oeffnet und ein paar Kindprozesse (standardmaessig 5 an der Zahl) unter dem User ``www-data`` forked, die als *Worker* die Client-Anfragen beantworten. Diese Kindprozesse werden je nach Auslastung vom Mutterprozess gespawned oder gekilled. Parameter, wie die initiale Anzahl an gestarteten Kindprozessen bei Serverstart, koennen ueber Direktiven in der bekannten ``apache2.conf`` festgelegt werden.

(screenshot apache2 ps -aux)

**Bemerkung**: Der User ``www-data`` wird bei der Apache-Installation erstellt und ist ein Systemuser, sprich ohne Homeverzeichnis. Der Vorteil von einen neuen User ist, dass die Rechte individuell pro Service/Daemon anpassbar sind und kein Service ausserhalb seiner Berichtigungsgrenzen arbeitet.

TLS
###

Bei TLS (Transport Layer Security, auch unter der Vorgaengerbezeichnung SSL bekannt) handelt es sich um ein Verschlüsselungsprotokoll in der OSI-Schicht 5 (Sitzungsschicht). Durch seinen erweiternden Charakter kann es verwendet werden um Protokolle hoeherer Schichten transparent zu verschluesseln. Am Beispiel von HTTP und HTTPS wird in beiden Faellen das HTTP-Protokoll verwendet, nur einmal mit der zusaetzlichen Sicherungsschicht realisiert durch TLS.

Funktionsweise
**************

Der Client startet einen Verbindungsversuch zum Server. Letzterer reagiert, indem er mit seinem eignenen Zertifikat antwortet. Der Client ueberprueft das Zertifikat und stellt sicher, dass der Servername mit dem im Zertifikat uebereinstimmt. Per assymetrischer Verschluesselung wird ein symmetrischer Schluessel ausgetauscht, der in der Sitzung zur Verschluesselung der Nutzdaten in Zukunft verwendet wird.

Im Fall von namensbasiertem virtuellen Hosting mit HTTPS gibt es eine Besonderheit zu beachten:

Bei HTTPS muss der Webserver fuer jeden Hostnamen ein eignenes Zertifikat ausliefern. Der Hostname ist dem Apache-Server aber erst nach dem TLS-Handshake bekannt. Eine Loesung besteht in der Erweiterung des TLS-Protocols um den Mechanismus Server Name Indication (SNI), welches seit TLS Version 1.2 verfuegbar ist. Hierbei wird die Hostname-Information bereits waehrend des TLS-Handshakes an den Apache-Server uebermittelt, sodass dieser das entsprechende Zertifikat zurueckgeben kann.

Exercises
#########

Einrichtung des Apache Webservers und erste Schritte
****************************************************
Zunächst wird der Apache Webserver über die Paketverwaltung mit dem Befehl ``sudo apt-get install apache2`` installiert.

Eine eigene ``index.html`` mit folgendem Content wurde im Default-Verzeichnis ``/var/www/html`` angelegt:

::

    <html>
        <head>
            <title>testpage</title>
        </head>
        <body>
            testcontent
        </body>
    </html>

Wenn man ``sdi1b.mi.hdm-stuttgart.de`` im Browser aufruft, erscheint wie erwartet unsere Testseite.

.. image:: images/Apache/01_customIndexHTML.png

Benannt man die ``index.html`` in ``doc.html`` um, erscheint die IndexOf-Seite, da der Einstiegspunkt einer ``index.html``-Datei nicht mehr vorhanden ist.

.. image:: images/Apache/02_renamedToDocHTML.png

Installation von ``apache2-doc`` sowie Suche der URL
****************************************************
Installiert werden kann die Apache Doku mit dem Command ``sudo apt-get install apache2-doc``.

Verstaendnis 1:
Die URL des Repositories finden, von dem das Package ``apache2-doc`` heruntergeladen wird. Das geht nicht mit dem in der Aufgabe erwaehnten Tipp "dpkg...", sondern geht ueber den Command ``apt-cache policy apache2-doc``, welcher die URLs wie folgt ausgibt:

::

    apache2-doc:
      Installed: 2.4.7-1ubuntu4.4
      Candidate: 2.4.7-1ubuntu4.4
      Version table:
     *** 2.4.7-1ubuntu4.4 0
            500 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
            500 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packag  es
            100 /var/lib/dpkg/status
         2.4.7-1ubuntu4 0
            500 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
 
Verstaendnis 2:
Den Pfad finden, ueber den der Apache Webserver die installierte Doku zur Verfuegung stellt. Laut Tipp ist ein Hinweis in einer Datei im ``apache2-doc``-Package zu finden. Mit dem Command ``dpkg -L apache2-doc`` lassen sich nun alle zum Packe zugehoerigen Dateien samt absolutem Pfad ausgeben. Die Ausgabe ist jedoch zu komplex und kann mit dem grep-Filter entsprechend reduziert werden. Eine uebersichtlichere Ausgabe laesst sich mit dem Befehl ``dpkg -L apache2-doc | grep -vE '(manual|examples)'`` erzeugen:

::

    /.
    /usr
    /usr/share
    /usr/share/doc
    /usr/share/doc/apache2-doc
    /usr/share/doc/apache2-doc/copyright
    /usr/share/doc/apache2-doc/changelog.Debian.gz
    /usr/share/doc-base
    /etc
    /etc/apache2
    /etc/apache2/conf-available
    /etc/apache2/conf-available/apache2-doc.conf

Wie zu sehen ist, wurden die in Frage kommenden Files erheblich reduziert. Die einzigste Datei, die Sinn macht, ist die ``/etc/apache2/conf-available/apache2-doc.conf``. Ein Apache-Kenner haette sofot in dieser Datei nachschauen koennen, da in diesem Verzeichnis alle Konfigurationsdateien von auf Apache beruhenden Packages, also z.B. der Apache-Doku, dem MySql-Frontend und dem Nagios-Frontend, gehalten werden.

Die gefundene Datei enthaelt:

::

    Alias /manual /usr/share/doc/apache2-doc/manual/
    
    <Directory "/usr/share/doc/apache2-doc/manual/">
        Options Indexes FollowSymlinks
        AllowOverride None
        Require all granted
        AddDefaultCharset off
    </Directory>

In dieser Datei sind 2 Pfade zu sehen:
* ``/usr/share/doc/apache2-doc-manual``: Der absolute Pfad, auf dem die Apache-Doku auf dem Server liegt.
* ``/manual``: Ein relativer Pfad als Alias, ueber den die Doku im Browser aufgerufen kann. In unserem Fall waere das also ``sdi1b.mi.hdm-stuttgart.de/manual``.

Ruft man die Seite ``sdi1b.mi.hdm-stuttgart.de/manual`` im Browser auf, erscheint erwartungsgemaess die Apache-Doku:

.. image:: images/Apache/03_apacheDocSlashManual.png

Auffaellig ist, dass beim Browsen dieser URL eine automatische Weiterleitung nach ``sdi1b.mi.hdm-stuttgart.de/manual/en/index.html`` erfolgt. Diese Weiterleitung wird von einer ``index.html`` im ``/manual``-Verzeichnis angestossen.

SDI-Doku hochladen und zugaenglich machen
*****************************************
Die SDI-Doku besteht aus mehreren Files, daher macht es Sinn die Doku vor dem Upload in eine Datei zu packen. Somit muss man nur eine Datei manuell hochladen. Gepackt wurde die Doku in einen Tarball mittels ``tar -cvzf sphinxdoku.tgz html`` (**ERKLAERUNG DER PARAMETER**)). Die Uebertragung von lokalem PC auf den Server ist mit dem Tool ``scp`` realisierbar, konkret dem Befehl ``scp sphinxdoku.tgz root@141.62.75.106:.`` (**ERKLAERUNG DER PARAMETER**). Durch die Angabe des Punkts hinten, landet die Datei dann serverseitig im Homeverzeichnis des Users root. Anschliessend muss die Datei wieder entpackt werden, z.B. mit dem Befehl ``tar -xvf sphinxdoku.tgz``. Unsere SDI-Doku liegt nun also auf dem Server in dem Verzeichnis ``/home/sdidoc/``.

Nun muss der Apache entsprechend konfiguriert werden, damit die Doku auch ueber einen Browser erreichbar ist:

::

    <Directory /home/sdidoc/>
           Options Indexes FollowSymLinks
           AllowOverride None
           Require all granted
    </Directory>
 
Es gibt 2 Moeglichkeiten:  Eine Redirect-Directive oder einen Alias. Vorraussetzung fuer beide Varianten ist, dass im SDI-Doku-Verzeichnis eine ``index.html`` als Einstiegspunkt existiert, was bei uns von unserem Doku-Tool Shinx bereits so erstellt wurde.

``Alias``-Direktive:

Alias wurden im Prinzip schon in der letzten Aufgabe rund um ``apache2-doc`` behandelt. Die Alias-Direktive nimmt einen relativen Pfad (relativ zum ServerName), also ``/mh203``, entgegen und mappt diesen auf einen anderen Pfad, in unserem Fall also ``/home/sdidoc``.
::

    <VirtualHost *:80>
            ServerName sdi1b.mi.hdm-stuttgart.de
            DocumentRoot /var/www/html
            Alias /mh203 /home/sdidoc
            <Directory /home/sdidoc>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
            </Directory>
    </VirtualHost>

Wie folgender Screenshot zeigt, funktioniert dieser Ansatz:

.. image:: images/Apache/04_sdiDocSlashMH203.png

``Redirect``-Direktive:

Hierbei wird die Anfrage nach ``sdi1b.mi.hdm-stuttgart.de/mh203`` auf einen anderen Host, also wie in diesem Beispiel auf ``sdidoc.mi.hdm-stuttgart.de``, weitergeleitet. Der Client muss dabei eine neue HTTP-Anfrage an die neue URL schicken. Demnach gibt es in der Apache-Konfigurationsdatei auch zwei ``VirtualHost``-Eintraege, einen fuer die Weiterleitung, den anderen fuer den eigentlichen Aufenthalt der SDI-Doku auf ``sdidoc.mi.hdm-stuttgart.de``.

**Bemerkung**: Der virtuelle Host ``sdidoc.mi.hdm-stuttgart.de`` muss vom DNS-Server korrekt aufgeloest werden. Auf meinem Server habe ich daher dieses Domainnamen in meine Zonefile des DNS-Servers mit aufgenommen, sodass dieser auf die IP 141.62.75.106 aufgeloest wird. Vergleiche auch mit naechster Aufgabe.

::

    <VirtualHost *:80>
            ServerName sdi1b.mi.hdm-stuttgart.de
            DocumentRoot /var/www/html
            Redirect /mh203 http://sdidoc.mi.hdm-stuttgart.de
    </VirtualHost>
    <VirtualHost *:80>
            ServerName sdidoc.mi.hdm-stuttgart.de
            DocumentRoot /home/sdidoc/
            <Directory /home/sdidoc>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
            </Directory>
    </VirtualHost>

Auch dieser Ansatz funktioniert, wenn der DNS-Eintrag fuer ``sdidoc.mi.hdm-stuttgart.de`` eingetragen ist:

.. image:: images/Apache/05_sdiDocSubdomain.png

Einrichtung von virtuellen Hosts
********************************
Die Konfigurationsdatei, mit der das Verhalten erzielt werden kann sieht folgendermassen aus:

::

    <VirtualHost *:80>
           ServerAdmin webmaster@localhost
           DocumentRoot /var/www/html
           ErrorLog ${APACHE_LOG_DIR}/error.log
           CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    <VirtualHost *:80>
            ServerName mh203.mi.hdm-stuttgart.de
            DocumentRoot /home/sdidoc
            <Directory /home/sdidoc>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
            </Directory>
    </VirtualHost>
    <VirtualHost *:80>
            ServerName manual.mi.hdm-stuttgart.de
            DocumentRoot /usr/share/doc/apache2-doc/manual/
    </VirtualHost>

Die eigene ``index.html`` mit dem Inhalt ``testcontent`` ist weiterhin ueber ``sdi1b.mi.hdm-stuttgart.de`` erreichbar (erster VirtualHost-Eintrag). Ein ServerName muss nicht zwangsweise mit angegeben werden, denn so wird dieser VirtualHost fuer alle Anfragen verwendet, die nicht einen anderen ServerName anfragen (s. folgende VirtualHosts), eine Art Fallback also. Der zweite VirtualHost-Eintrag ermoeglicht den Zugriff auf die SDI-Doku ueber ``mh203.mi.hdm-stuttgart.de``, der dritte Eintrag auf die Apache-Doku ueber ``manual.mi.hdm-stuttgart.de``. Ersteren muss man wieder ueber die ``Directory``-Direktive erweitern, sodass das Verzeichnis ``/home/sdidoc`` zugaenglich ist.

**Bemerkung**: Auch hier wieder: die beiden Subdomains muessen in die Zonesfile des DNS-Servers aufgenommen werden, damit diese Namen auf die IP des Servers (141.62.75.106) verweisen. DNS-Serverneustart mit ``service bind9 restart``. 

Damit auch der eigene DNS-Server zur Aufloesung verwendet wird, muss unter Ubuntu dieser manuell eingetragen werden. Das Ziel ist, dass in der Datei ``/etc/resolv.conf`` unser eigener DNS-Server an erster Stelle steht. Dazu kann der Eintrag in ``/etc/resolvconf/resolv.conf.d/head`` hinzugefuegt werden. Hintergrund ist, dass die ``/etc/resolv.conf`` aus den beiden ``head``- und ``base``-Dateien generiert wird. Der Inhalt von ``head`` wird bei der Generierung immer vor dem von ``base`` in das resultierende File eingefuegt.

*Quelle: http://askubuntu.com/questions/157154/how-do-i-include-lines-in-resolv-conf-that-wont-get-lost-on-reboot*

Wir fuegen also den Eintrag in die ``head``-Datei ein:

::

    # Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
    #     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
    nameserver 141.62.75.106

Die Warnung steht am Anfang dort, weil diese den User davon bewahren soll, die generierte ``resolv.conf`` zu aendern. In unserem Fall koennen wir die Warnung ignorieren. Mit dem Befehl ``sudo resolvconf -u`` kann ``resolv.conf`` neu generiert werden. Das Resultat in ``resolv.conf``:

::

    # Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
    #     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
    nameserver 141.62.75.106
    nameserver 127.0.1.1

Wie zu sehen ist, steht unser DNS-Server an erster Stelle, gefolgt von Nameserver des Host-OS (Ubuntu laeuft hier in einer VM als Guest-OS).


SSL-Einrichtung
***************

LDAP Authentifizierung
**********************

