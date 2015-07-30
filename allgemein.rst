*********
Allgemein
*********

Die vorliegende Dokumentation wurde im Rahmen der Vorlesung **Software Defined Infrastructure** an der **Hochschule der Medien** erstellt. Gegenstand der Vorlesung waren die Themen 

1. Genereller Umgang mit Linux
2. LDAP
3. DNS
4. Webserver
5. Samba
6. Nagios

Für jedes dieser Themengebiete wurden in Gruppen à 2-3 Studenten Aufgaben bearbeitet und die Vorgehensweise zur Lösung der Problemstellungen auf dieser Webseite dokumentiert.


Einrichtung der Arbeitsumbebung
*******************************

Für die Bearbeitung der Aufgaben stand jedem Studenten eine eigene virtuelle Maschine (VM) zur Verfügung, auf die mit SSH zugegriffen wurde. Ursprünglich war der Zugriff auf alle virtuellen Maschinen mit demselben Passwort "gesichert". Die erste Aufgabe war es eine Public-Key-Authentifizierung einzurichten.

Auf den Poolrechnern wurde zu diesem Zweck zunächst ein 4096 bit-Schlüsselpaar mit dem Befehl ``ssh-keygen -b 4096`` erzeugt. Während dieses Vorgangs musste ein Passwort zur Sicherung des Private Keys sowie deren Dateiname angegeben werden. Die Schlüssel müssen in dem Verzeichnis ``/home/.../.ssh/`` vorliegen. D. h. entweder müssen die erzeugten Schlüssel dort hin kopiert werden, oder der Befehl von oben muss in diesem Verzeichnis ausgeführt werden.  Als Name sollte **id_rsa** gewählt werden, damit der SSH-Client diesen automatisch verwendet. Andernfalls muss der Pfad zu dem Schlüssel bei der Anmeldung manuell angegeben werden: ``ssh user@server -i /pfad/zum/schluessel``.

::

    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/.../.ssh/id_rsa): id_rsa
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/.../.ssh/id_rsa.
    Your public key has been saved in /home/.../.ssh/id_rsa.pub.
    The key fingerprint is:
    ...
  
Der *Public Key* kann beliebig verteilt werden. Der *Private Key* soll dazu dienen, den Ersteller eindeutig zu identifizieren. Der Beweis für dessen Identität ist der **Besitz** des Private Key sowie die **Kenntnis** des Passworts. 

Anschließend wurde der Public Key per SCP in die ``authorized_keys``-Datei im ``~/.ssh``-Verzeichnis der jeweiligen VM kopiert. In dieser Datei sind die Schlüssel aller für den SSH-Zugriff berechtigten Benutzer hinterlegt.

Da die Authentifizierung fortan ausschließlich mit Schlüsseln erfolgen sollte, wurde die Passwort-Authentifizierung gänzlich deaktiviert. Hierfür wurde der entsprechende Eintrag in der Konfigurationsdatei ``/etc/ssh/sshd_config`` editiert:

::

    PasswordAuthentication no
    

Für die Verwendung von SSH war anschließend keine Passworteingabe mehr nötig.

::

    ssh root@sdi2b.mi.hdm-stuttgart.de
    Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 2.6.32-19-pve x86_64)
    
     * Documentation:  https://help.ubuntu.com/
    Last login: Thu Jul 30 07:15:37 2015 from 192.168.222.46  
    root@sdi2b:~#
