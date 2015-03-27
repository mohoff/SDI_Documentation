

****
LDAP
****

The LDAP concept
################

SOMETHING AMNOG THOSE LINES:

- schemas (also built-ins like cosine, nis, inetorgperson)
- tree structure

-  A LDAP directory is a tree of data entries that is hierarchical in nature and is called the Directory Information Tree (DIT).
- An entry consists of a set of attributes.
- An attribute has a type (a name/description) and one or more values.
- Every attribute must be defined in at least one objectClass.
- Attributes and objectclasses are defined in schemas (an objectclass is actually considered as a special kind of attribute).
- Each entry has a unique identifier: its Distinguished Name (DN or dn). This, in turn, consists of a Relative Distinguished Name (RDN) followed by the parent entry's DN.
- The entry's DN is not an attribute. It is not considered part of the entry itself.
- The terms object, container, and node have certain connotations but they all essentially mean the same thing as entry, the technically correct term.
- For example, below we have a single entry consisting of 11 attributes where the following is true:
- DN is "cn=John Doe,dc=example,dc=com"
- RDN is "cn=John Doe"
- parent DN is "dc=example,dc=com"

_ Abkuerzungen:

* dc
* dn (distinguished name)
* rdn (relative distinguished name)
* cn
* ou


Setting up a LDAP-Server
########################

Man unterscheidet zwischen dem OpenLDAP server daemon im Package ``slapd`` und LDAP
management utilities im Package ``ldap-utils``.

::

  sudo apt-get install slapd ldap-utils

Waehrend der Installation muss man Admin credentials festlegen, die fuer den
``rootDN`` der LDAP-Datenbank gesetzt werden, by default ist das

::

  cn=admin,dc=mi,dc=hdm-stuttgart,dc=de

Neben der eigentlichen LDAP-Datenbank, in der spaeter Daten gespeichert werden, wird
eine config-database erstellt, auf die man sich auf eine andere Art und Weise (TODO)
authentifizieren muss.

Die Installation erstellt eine lauffaehige Konfiguration, darunter eine Datenbank
in der die LDAP-Daten gespeichert werden koennen.

Der "base DN" (DN = Distinguished Name) dieser Instanz wird vom Domainnamen des
localhosts abgeleitet. Alternativ kann man die Datei ``/etc/hosts`` editieren, um manuell
Domainnamen fuer localhost vergeben, sodass ein erwuenschter base DN erstellt
werden kann. Die default Konfiguration in unseren VM ist daher

::

  dc=mi,dc=hdm-stuttgart,dc=de

Die ``config``-Datenbank
************************

Der Inhalt der config-Datenbank sieht aus wie folgt:

.. code-block:: html
  :linenos:

  /etc/ldap/slapd.d/
  /etc/ldap/slapd.d/cn=config
  /etc/ldap/slapd.d/cn=config/cn=module{0}.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={0}core.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={1}cosine.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}nis.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema/cn={3}inetorgperson.ldif
  /etc/ldap/slapd.d/cn=config/cn=schema.ldif
  /etc/ldap/slapd.d/cn=config/olcBackend={0}hdb.ldif
  /etc/ldap/slapd.d/cn=config/olcDatabase={0}config.ldif
  /etc/ldap/slapd.d/cn=config/olcDatabase={-1}frontend.ldif
  /etc/ldap/slapd.d/cn=config/olcDatabase={1}hdb.ldif
  /etc/ldap/slapd.d/cn=config.ldif

Direkte Aenderungen in der config-Datenbank sind nicht empfohlen, man soll besser
ueber das LDAP Protocol (Tool aus dem Package ``ldap-utils``) Aenderungen vornehmen.

The LDAP-Protocol
#################
Befehl ``ldapsearch``:
::

  [sudo] ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn

Variation davon:
::

  [sudo] ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn

Entweder ohne Authentifizierung (Parameter ``-x``) oder mit "Simple Authentication
and Security Layer" (SASL) (-Y <SASL mechanism>).

.. topic:: ``ldapsearch``

  .. glossary::
    ``-Q``
      Use SASL Quiet mode. Never prompt.

    ``-LLL``
      Displaying: restricts output to LDIFv1, hides comments, disables
      printing of the LDIF version (each "L" restricts output more)

    ``-Y <mechanism>``
      Authentication: specifies the authentication mechanism. Common ones are ``DIGIEST-MD5``, ``KERBEROS_V4`` and ``EXTERNAL``.
      Here: ``EXTERNAL`` which enables authentication over a lower level security mechanism like TLS.

    ``-h <URIs>``
      Specify URI(s) referring to the LDAP server(s). Default is ``ldap:///``
      which implies LDAP over TCP. Used ``ldapi:///`` also uses the protocol LDAP but uses IPC
      (UNIX-domain socket) instead of TCP.

    ``-b <searchbase>``
      Specify a searchbase as the starting point for the search. In our
      case ``cn=config``

    ``-x``
      Use simple authentication instead of SASL.

    ``<filter>``
      Specifies an output filter. If not specified, the default filter ``(objectClass=*)``
      is used. We used ``dn``, so all distinguished names inside the searchbase will be displayed


LDIF Files
##########

add data

Apache Directory Studio
#######################

Ein auch als Eclipse-Plugin erhaeltliches Tool, mit dem CRUD-Operations auf LDAP-Datenbaenken
ausgefuehrt werden koennen.
