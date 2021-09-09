# Troubleshooting

TODO: intro, motivatie

## Bottom-up troubleshooting

Wanneer je problemen met een netwerkservice moet troubleshooten is het essentieel om een **systematische en grondige** benadering te volgen. Als model voor een systematische aanpak gebruiken we hier de TCP/IP protocolstack. We doorlopen die van de onderste laag naar boven en controleren op elke laag alle nodige instellingen. Het is belangrijk om deze volgorde aan te houden en geen stappen over te slaan. Als een netwerkdienst niet beschikbaar is voor gebruikers heeft het geen zin om de firewall-configuratie te controleren, als er een probleem is met de routering naar de server.

| Laag                | Protocols                     | Sleutelwoorden            |
| :------------------ | :---------------------------- | :------------------------ |
| Applicatielaag      | HTTP, DNS, SMB, FTP, SSH, ... |                           |
| Transportlaag       | TCP, UDP                      | sockets, poorten          |
| Internetlaag        | IP, ICMP                      | routering, IP-adressering |
| Netwerktoegangslaag | Ethernet                      | switch, MAC-adressering   |
| Fysieke laag        |                               | UTP-kabel                 |

## Enkele algemene richtlijnen

Voor we de nodige controles op elke laag overlopen, eerst enkele algemene best-practices:

- Stel een **checklist** samen en houd deze up-to-date terwijl je meer ervaring opdoet. Dit is belangrijk! De kan is klein dat je elk commando uit het hoofd kent en als je met netwerkproblemen te kampen hebt, kan je niet gaan Googlen...
- Wees **systematisch** en volg dezelfde stappen in de juiste volgorde.
- Wees **grondig**, en sla geen stappen over (bijvoorbeeld het controleren van de kabels), tenzij bewezen is dat alles op dat niveau werkt zoals verwacht.
- Werk in **kleine stappen** en controleer bij elke wijziging of deze het verwachte resultaat oplevert.
- Maak **geen veronderstellingen**. Controleer.
- Bewaar een **reservekopie** van originele configuratiebestanden en de laatste "werkende" versie.
- **Valideer** altijd de syntaxis van configuratiebestanden voordat je ze toepast.
- Controleer de (juiste) **systeemlogbestanden**. Leer jezelf aan hoe je efficiënt moet zoeken in de systeemlogs.
- Open een aparte terminal die de **logs in realtime** toont (bijvoorbeeld journalctl -f)
- **Automatiseer je tests** (met een Shell-script, [BATS-testscript](https://github.com/bats-core/bats-core), Ansible-playbook, Serverspec, etc.)
- **Foutmeldingen** geven meestal een idee van waar je moet zoeken en kunnen een snelkoppeling zijn in het probleemoplossingsproces. Zoek naar de foutmelding in de logboeken of gegenereerd door clientsoftware en weet hoe je deze moet interpreteren. u

En tenslotte:

- **Stop met pingen naar www.google.com!** Dit is geen goede test, want je hebt er niets aan als dit niet lukt. Er moeten teveel voorwaarden vervuld zijn eer dit commando kan slagen.

## Fysieke en netwerktoegangslaag

In deze fase controleren we de netwerkhardware, met name netwerkkabels en poorten.

- Staat de stroom aan?
- Is de netwerkkabel aangesloten?
    - Ga in VirtualBox naar de VM-instellingen, Netwerk, selecteer de actieve interfaces, klik op "Geavanceerd" en zorg ervoor dat het selectievakje "Kabel aangesloten" is aangevinkt.
    - Gebruik bij een fysieke UTP-kabel een kabeltester om te controleren of deze goed werkt
- Branden de LED's van de Ethernet-poort, zowel op de machine als op de switch?
    - Zo niet, test dan de kabel
    - Sommige schakelaars hebben een andere kleur voor b.v. FastEthernet (100 Mbps) en Gigabit Ethernet. Controleer of je de verwachte kleur ziet.
- Gebruik het commando `ip-link`
    - UP: ok, de interface is aangesloten
    - NO-CARRIER: geen signaal op deze interface

Kabel is correct aangesloten:

```console
$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s31f6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 20:2f:b9:f1:cc:84 brd ff:ff:ff:ff:ff:ff
```

Kabel is **niet** correct aangesloten:

```console
$ ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s31f6: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 20:2f:b9:f1:cc:84 brd ff:ff:ff:ff:ff:ff
```

## Internetlaag

De internetlaag is verantwoordelijk voor het routeren van netwerkverkeer tussen hosts op het netwerk. Normaal krijgt een apparaat dat aansluit op een lokaal netwerk automatisch instellingen toegekend van een DHCP-server. Servers hebben soms een vast IP-adres en daar moet je dan toch deze instellingen controleren. Werk altijd in deze volgorde:

1. Controleer de lokale netwerkinstellingen op het apparaat
2. Controleer routering binnen het lokaal netwerk
3. Controleer routering naar het internet

### Lokale netwerkinstellingen

Om te kunnen communiceren, moet elke host *drie* instellingen correct hebben geconfigureerd:

1. Aan de netwerkinterface moet een **IP-adres** zijn toegewezen
2. Er moet een **standaardgateway** zijn ingesteld
3. Er moet een **DNS-server** zijn ingesteld

Het IP-adres kan automatisch (DHCP) of handmatig zijn ingesteld. Controleer dit in `/etc/sysconfig/network-scripts/ifcfg-IFACE` (met IFACE de naam van de netwerkinterface).

Dit is een fragment uit een netwerkinterfaceconfiguratiebestand dat is geconfigureerd om DHCP te gebruiken:

```bash
DEVICE="enp0s3"
ONBOOT="yes"
BOOTPROTO="dhcp"
```

Dingen om te controleren:

- `DEVICE` moet de interfacenaam zijn
- `ONBOOT` moet `yes` zijn, anders zal het systeem niet proberen de interface te activeren bij het opstarten of bij het herstarten van de netwerkinterfaces (`systemctl restart network`)
- `BOOTPROTO` moet zijn ingesteld op `dhcp`

Als er een vast IP-adres is ingesteld, kan de configuratie er als volgt uit zien:

```bash
DEVICE=enp0s8
ONBOOT=yes
BOOTPROTO=none
IPADDR=192.168.56.24
NETMASK=255.255.255.0
```

- `DEVICE` en `ONBOOT` zijn ingesteld zoals in het vorige voorbeeld
- `BOOTPROTO` zou `none` moeten zijn
- `IPADDR` en `NETMASK` moeten gespecifieerd zijn

#### IP-adres controleren

Gebruik `ip address` (of de korte versie `ip a`) om de IP-adressen voor elke interface weer te geven.

Je moet de verwachte waarde kennen! Indien je het exacte IP-adres kent moet je in ieder geval weten wat het netwerkbereik of netwerk-IP en netwerkmasker is van het LAN waarin de host zich bevindt.

- Veel thuisrouters hebben een IP-bereik van 192.168.0.0/24 of 192.168.1.0/24
- Op een VirtualBox NAT-interface moet het IP-adres van de VM 10.0.2.15/24 zijn. Let hier op het netwerkasker! Normaal is dat /8 voor dit IP-adres!
- Op een VirtualBox host-only interface hangt het IP-adres van de VM af van hoe je dit netwerk hebt geconfigureerd. Het *standaard* host-only netwerk heeft IP-bereik 192.168.56.0/24, met een DHCP-server die adressen uitdeelt vanaf 192.168.56.101 t/m 192.168.56.254.

Mogelijke problemen en oorzaken (in geval van automatische IP-toewijzing met DHCP):

- Geen IP:
    - De DHCP-server is onbereikbaar
    - De DHCP-server geeft geen IP aan deze host (bv. als gewerkt wordt met gereserveerde IP-adressen op basis van MAC-adres)
- Het IP-adres ziet eruit als 169.254.x.x
    - Er kon geen DHCP-server worden bereikt en er werd een "link-local"-adres toegewezen (typisch voor Windows-systemen)
- Het IP-adres niet in het verwachte bereik
    - Mogelijk heb je voordien per ongeluk een vast IP-adres ingesteld en ben je vergeten het opnieuw op DHCP in te stellen...
    - Misschien is de DHCP-server verkeerd ingesteld

Mogelijke problemen en oorzaken (in geval van handmatige IP-instelling):

- Het IP-adres ligt niet in het verwachte bereik
    - Fout bij toewijzing van het IP-adres, controleer het configuratiebestand
- Correct IP-adres, maar je krijgt de foutboodschap "network unreachable"
    - Controleer het netwerkmasker, dit moet identiek zijn voor alle hosts op het LAN!

#### Default Gateway

Gewoonlijk is een host via een switch verbonden met een LAN. Netwerkverkeer naar de buitenwereld gaat via een router, aangesloten op hetzelfde LAN. Elke host op het LAN zou deze router, de "default gateway", moeten kennen.

Gebruik het commando `ip route` (of korte versie `ip r`). Er zou een regel moeten zijn die begint met `default via x.y.z.w`.

Mogelijke problemen en oorzaken (in geval van automatische IP-toewijzing met DHCP):

- Er is geen standaard gateway ingesteld
    - er is hoogstwaarschijnlijk ook een probleem met het IP-adres van deze host, los dit eerst op
    - Als je de DHCP-server beheert, is deze misschien slecht geconfigureerd?
- Onverwacht standaard gateway-adres
    - Mogelijk heb je de standaardgateway eerder handmatig ingesteld en ben je vergeten deze opnieuw op DHCP in te stellen
    - Als je de DHCP-server beheert, is deze misschien slecht geconfigureerd?

#### DNS-server

Om het Internet te kunnen gebruiken, heeft elke host toegang nodig tot een DNS-server om de IP-adressen die bij hostnamen (bv. www.hogent.be) horen te kunnen opzoeken. Bekijk het bestand `/etc/resolv.conf`. Het heeft meestal een header die vermeldt dat het automatisch is gegenereerd, en zou een of meer regels moeten hebben die beginnen met `nameserver`.

```console
$ cat /etc/resolv.conf
# Generated by NetworkManager
zoek hogent.be
naamserver 193.190.173.1
naamserver 193.190.173.2
```

Voor elke hostnaam die je vanop deze machine wil bereiken, zal het systeem een query sturen naar één van de opgegeven nameservers om het IP-adres op te vragen.

Op recente versies van enkele Linux-distributies (bv. Ubuntu, Fedora) zal je het volgende zien:

```console
$ cat /etc/resolv.conf
# This file is managed by man:systemd-resolved(8). Do not edit.
[...]
nameserver 127.0.0.53
```

Dan is `systemd-resolved` geïnstalleerd. Die simuleert zelf het gedrag van een DNS-server die antwoordt op DNS-queries. `systemd-resolved` zal dan de vraag doorgeven aan een externe DNS-server. Welke dat is kan je opvragen met `resolvectl`:

```console
$ resolvectl
Global
       Protocols: LLMNR=resolve -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (enp0s31f6)
    Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6                                   
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.0.1
       DNS Servers: 192.168.0.1
```

In dit voorbeeld is dat 192.168.0.1, wellicht een thuisrouter die ook dienst doet als DNS forwarder.

### Controleer routering binnen het LAN

Als de hierboven opgesomde instellingen correct zijn, kan je controleren of andere hosts op het LAN bereikbaar zijn.

- Ping de standaard gateway: `ping a.b.c.d`
- Ping een andere bekende host op het LAN
- Als dit werkt kan je proberen een host buiten het LAN te pingen

**Houd er rekening mee** dat `ping` (en andere hulpprogramma's voor het oplossen van netwerkproblemen zoals de `traceroute`-familie) niet altijd werken. Sommige systeembeheerders zullen ICMP-verkeer op routers blokkeren, waardoor de resultaten onbruikbaar worden.

Een commando als `ping www.google.com` (voor sommigen het eerste commando dat ze proberen in geval van netwerkverbindingsproblemen) is niet erg geschikt, omdat het van te veel dingen tegelijk afhangt:

- de netwerkinstellingen van de host moeten correct zijn
- routering zou moeten werken
- DNS zou beschikbaar moeten zijn
- geen enkele router tussen deze host en het doel mag ICMP blokkeren
- enz.

Een Windows-systeem kan ook ICMP-verkeer blokkeren (inclusief `ping`). Zorg ervoor dat je de firewall-instellingen controleert en indien nodig dit type netwerkverkeer toestaat. De volgende PowerShell-oneliner doet precies dat:

```powershell
Get-NetFirewallRule -DisplayName "*Echo Request*" | Set-NetFirewallRule -enabled true
```

### Controleer beschikbaarheid van de DNS service

Het is niet omdat er een DNS-server geconfigureerd is in `/etc/resolv.conf` of via `systemd-resolved` dat deze ook beschikbaar is voor clients en antwoordt op requests. Je kan controleren of de DNS-service beschikbaar is met `nslookup` (dit commando ken je allicht van Windows), `dig` of `getent ahosts`.

```console
$ nslookup www.hogent.be
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
www.hogent.be	canonical name = hogent.be.
Name:	hogent.be
Address: 193.190.173.132
```

In dit voorbeeld stuurt `nslookup` de query naar de DNS-server gespecifieerd in `/etc/resolv.conf`. Je kan ook een specifieke DNS-server ondervragen:

```console
$ nslookup www.hogent.be 9.9.9.9
Server:		9.9.9.9
Address:	9.9.9.9#53

Non-authoritative answer:
www.hogent.be	canonical name = hogent.be.
Name:	hogent.be
Address: 193.190.173.132
```

Het commando `dig` is veel uitgebreider en hier kan je ook andere types DNS-queries uitvoeren (NS, MX, AXFR, enz.):

```console
$ dig www.hogent.be @9.9.9.9 +short
hogent.be.
193.190.173.132
```

Hiermee vraag je het IP-adres van www.hogent.be aan DNS-server 9.9.9.9 en drukt het resultaat in een bondige vorm af. Als je `+short` weglaat krijg je meer informatie te zien. Zie het onderdeel BIND troubleshooting voor meer info.

Op servers en systemen met een minimale installatie, is het commando `nslookup` of `dig` misschien niet geïnstalleerd. Als je met netwerkproblemen te kampen hebt, kan je het ook niet gauw installeren (en nieuwe software installeren op een productieserver is sowieso geen goed idee). In dat geval kan je gebruik maken van `getent ahosts`:

```console
$ getent ahosts www.hogent.be
193.190.173.132 STREAM hogent.be
193.190.173.132 DGRAM  
193.190.173.132 RAW 
```

Het nadeel van dit commando is dat je er enkel de in `/etc/resolv.conf` gespecifieerde DNS-server kan ondervragen, maar dat is natuurlijk beter dan niets.

Wanneer de DNS-server niet gespecifieerd is, of niet antwoordt op queries, kan je dezelfde zaken controleren als bij de default gateway.

## Transportlaag

In de transportlaag controleren we volgende zaken:

- Draait de service?
- Luistert de service op het juiste poortnummer?
- Is de firewall correct geconfigureerd?

We geven hier het voorbeeld van een webserver, maar je kan dit toepassen op elke service.

### Draait de service?

Gebruik hiervoor het commando `systemctl status`. We verwachten ergens in de uitvoer "active (running)" te zien.

```console
$ systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
       Docs: man:httpd.service(8)
```

In dit geval is de (Apache) webserver uitgeschakeld. We zien hier ook dat de service "disabled" is, wat betekent dat deze niet wordt opgestart bij het booten. Dit kan je oplossen met `sudo systemctl enable httpd`. We starten de service nu op en controleren het resultaat:

```console
$ sudo systemctl start httpd
[sudo] password for user: 
$ systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Fri 2021-06-25 21:46:10 CEST; 2s ago
       Docs: man:httpd.service(8)
   Main PID: 932880 (httpd)
     Status: "Started, listening on: port 80"
      Tasks: 213 (limit: 18723)
     Memory: 23.3M
        CPU: 100ms
     CGroup: /system.slice/httpd.service
             ├─932880 /usr/sbin/httpd -DFOREGROUND
             ├─932881 /usr/sbin/httpd -DFOREGROUND
             ├─932883 /usr/sbin/httpd -DFOREGROUND
             ├─932884 /usr/sbin/httpd -DFOREGROUND
             └─932885 /usr/sbin/httpd -DFOREGROUND

jun 25 21:46:09 nb1100380 systemd[1]: Starting The Apache HTTP Server...
jun 25 21:46:09 nb1100380 httpd[932880]: AH00558: httpd: Could not reliably determine the server's fully qualified domain nam>
jun 25 21:46:10 nb1100380 httpd[932880]: Server configured, listening on: port 80
jun 25 21:46:10 nb1100380 systemd[1]: Started The Apache HTTP Server.
```

Nu is de Apache server wel actief.

### Juiste poortnummers

Gebruik het commando `ss -tln` of `sudo ss -tlnp` (Show Sockets) om te controleren welke TCP-services (optie `-t`) op dit moment een open server-socket (`-l`) in gebruik hebben. Met de optie `-n` toon je de poortnummers in plaats van de service-naam (zoals gespecifieerd in `/etc/services`). De optie `-p` toont welk proces er luistert op deze server-socket, maar daar heb je root-rechten voor nodig (vandaar de `sudo`). Voor services die gebaseerd zijn op UDP geef je de optie `-u` in plaats van `-t`.

```console
$ ss -tln
State         Recv-Q        Send-Q                 Local Address:Port                 Peer Address:Port        Process        
LISTEN        0             4096                         0.0.0.0:5355                      0.0.0.0:*                          
LISTEN        0             4096                   127.0.0.53%lo:53                        0.0.0.0:*                          
LISTEN        0             128                          0.0.0.0:22                        0.0.0.0:*                          
LISTEN        0             4096                            [::]:5355                         [::]:*                          
LISTEN        0             511                                *:80                              *:*                          
LISTEN        0             128                             [::]:22                           [::]:*                          
```

Vooral de kolom `Local Address:Port` is nuttig. "Local Address" geeft aan op welke netwerkinterface de service aan het luisteren is:

- 127.0.0.1: enkel op de loopback-interface
- 0.0.0.0 of *: op alle interfaces
- [::]: op alle interfaces (IPv6)
- Als er een specifiek IP-adres opgegeven wordt, dan luistert de service enkel op de netwerkinterface met dat IP-adres

Verder moet je weten op welke poort je verwacht dat de service aan het luisteren is. Voor een webserver is dat typisch poort 80 (HTTP) en 443 (HTTPS). Poort 22 wordt normaal gebruikt door de Secure Shell daemon (`sshd`) en poort 53 door een DNS-server (in dit geval `systemd-resolved`, die ook poort 5353 gebruikt). Om zeker te zijn dat dit klopt, vragen we best ook de processen op:

```console
$ sudo ss -tlnp
State   Recv-Q Send-Q  Local Address:Port  Peer Address:Port  Process                                                                                                                       
LISTEN  0      4096          0.0.0.0:5355       0.0.0.0:*     users:(("systemd-resolve",pid=704,fd=12))                                                                                    
LISTEN  0      4096    127.0.0.53%lo:53         0.0.0.0:*     users:(("systemd-resolve",pid=704,fd=17))                                                                                    
LISTEN  0      128           0.0.0.0:22         0.0.0.0:*     users:(("sshd",pid=760464,fd=5))                                                                                             
LISTEN  0      4096             [::]:5355          [::]:*     users:(("systemd-resolve",pid=704,fd=14))                                                                                    
LISTEN  0      511                 *:80               *:*     users:(("httpd",pid=932885,fd=4),("httpd",pid=932884,fd=4),("httpd",pid=932883,fd=4),("httpd",pid=932880,fd=4))              
LISTEN  0      128              [::]:22            [::]:*     users:(("sshd",pid=760464,fd=7))                                                                                             
```

Als de service opgestart is, maar niet op het juiste poortnummer luistert, dan zal die ook niet beschikbaar zijn voor clients. Meestal moet je het poortnummer specifiëren in het configuratiebestand voor die service. Controleer dit, herstart de service na de nodige aanpassingen en controleer opnieuw met `sudo ss -tlnp`.

### Firewall-instellingen

Laat de firewall netwerkverkeer door naar de service? Controleer dit met `firewall-ctl`:

```console
$ sudo firewall-cmd --list-all
public (default, active)
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client mdns samba-client ssh http https
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

In dit geval zijn de services `http` en `https` bereikbaar. De services `dhcpv6-client mdns samba-client` staan op Fedora/RedHat altijd aan. Voor een server is `ssh` ook belangrijk en dit moet dus ook meestal aan staan.

Controleer steeds of:

- De netwerkinterface waarop de service luistert wordt vermeld in de lijn die begint met `interfaces:`
- De servicenaam wordt weergegeven.
    - De waarde moet er een zijn die wordt vermeld door `firewall-cmd --get-services`.
    - Merk op dat de servicenaam voor `firewalld` *niet noodzakelijk* gelijk is aan de `systemd` servicenaam. Bijv. de BIND service heet `named.service`, terwijl er in de firewall-instellingen naar wordt verwezen met `dns`.
- Als de servicenaam niet voorkomt in de uitvoer van `firewalld --get-services`, moeten de poortnummers en transportprotocol die door de service worden gebruikt worden vermeld (bijvoorbeeld bij gebruik van een niet-standaard poort of een service die niet bekend is onder 'firewalld').

Voeg zo nodig de service toe aan de firewall-configuratie en herstart:

```console
$ sudo firewall-cmd --add-service=http --permanent 
success
$ sudo firewall-cmd --add-service=https --permanent 
success
$ sudo firewall-cmd --reload
success
```

Vergeet hier de optie `--permanent` niet! Als je die weglaat, wordt de wijziging onmiddellijk doorgevoerd (wat op het eerste zicht is wat je wilt), maar na rebooten van het systeem of herstarten van de `firewalld.service`, zal deze tijdelijke wijziging ongedaan gemaakt worden en heb je terug de foute instellingen!

Als je een poortnummer moet specifiëren gebruik je volgende notatie:

```console
$ sudo firewall-cmd --add-port=9000/tcp --permanent 
success
$ sudo firewall-cmd --reload
success
```

Controleer na een wijziging altijd of de firewall-regels nu correct zijn!

TODO: kadertekstje over verwarring bij de uitspraak "open poorten".

## Applicatielaag

Specifieke zaken waar je moet op letten in deze laag hangen voor een groot stuk af van de service die je aan het configureren/troubleshooten bent. Apache, nginx, BIND, Vsftpd, Postfix, enz. hebben allemaal een eigen structuur en configuratie met onderling grote verschillen. Je moet de service in kwestie dus goed leren kennen!

Er zijn echter wel enkele algemene zaken die je in elke situatie kan controleren:

- De systeemlogs kunnen foutboodschappen bevatten die een indicatie zijn voor de oorzaak van het probleem.
- De meeste services hebben een tool waarmee je de syntax van het configuratiebestand kan controleren.
- De service moet beschikbaar zijn voor clients en antwoorden op requests/queries.

### Systeemlogbestanden

Controleer systeemlogs met `journalctl` of door het logbestand voor de service op te zoeken in `/var/log`. Alle systeemlogs die beheerd worden door `systemd-journald` kan je met dat eerste commando opvragen. Sommige applicaties worden niet beheerd of herkend door `systemd-journald` en die houden hun logs bij als een tekstbestand (in de directory `/var/log`). Het is nuttig om tijdens het hele troubleshooting-proces een aparte terminal te openen en daarin de relevante logs te openen. In de voorbeelden hieronder zorgt de optie `-f` (kort voor `--follow`) er voor dat het commando wacht op nieuwe log-events en die meteen toont wanneer die zich voordoen. Op die manier kan je meteen het effect zien van commando's, configuratiewijzigingen of andere handelingen die je op het systeem uitvoert.

- `sudo journalctl -f`: toont alle systeemlog-events die door `systemd-journald` ontvangen worden.
- `sudo journalctl -f -u httpd.service`: zal enkel de logs van de Apache server tonen.
- `sudo tail -f /var/log/httpd/error_log`: toont de inhoud van het tekstbestand `error_log`, waar soms foutboodschappen in terecht komen die je niet ziet met `journalctl`, zoals bijvoorbeeld de foutboodschappen van een PHP-applicatie.

### Configuratiebestanden

Controleer het configuratiebestand dat zich ergens in de directory `/etc/` moet bevinden. Voor Apache is dat bv. `/etc/httpd/httpd.conf`. Maak eerst een backup van de huidige toestand van het configuratiebestand voordat je wijzigingen aanbrengt!

- Valideer de syntaxis van het configuratiebestand, als dit voorzien is door de service.
    - Apache: `apachectl configtest`
    - BIND: `named-checkconf` en `named-checkzone`
    - Samba: `testparm`
    - Vsftpd: `vsftpd -olisten=NO`
    - enz.
- Verbeter waar nodig fouten in het configuratiebestand en herstart daarna de service, bv: `sudo systemctl restart httpd.service`.
    - Controleer in de logs of er nieuwe of andere foutboodschappen verschijnen
    - Controleer of de service nu wel draait (zie Transportlaag)

### Beschikbaarheid voor clients

Als we alle voorgaande stappen doorlopen hebben, en eventuele fouten weggewerkt hebben, dan mogen we op dit punt verwachten dat de service nu correct werkt en beschikbaar is voor clients. We kunnen dit soms op de server zelf controleren (via de loopback-interface), maar het is belangrijk om dit over het netwerk te doen. Zo komen soms nog onverwachte problemen naar boven. Netwerkverkeer op de loopback-interface wordt bijvoorbeeld nooit tegengehouden door de firewall, terwijl dat via fysieke interfaces wel kan gebeuren. Naast het gebruiken van de normale client-software om de service aan te spreken (bv. een webbrowser), zijn er nog enkele andere tools die van pas kunnen komen bij het onderzoeken van de interactie tussen clients en de service.

- Je kan een portscan uitvoeren met [nmap](https://nmap.org) om te controleren of de serverpoort bereikbaar is, bv.
    - `sudo nmap -sS -p 80,443 192.168.56.24`: voer een TCP SYN scan uit op poorten 80 en 443
- Gebruik een commandline-tool of test-commando om de service te ondervragen. Dit soort tools geven vaak betere foutboodschappen om de oorzaak van problemen te achterhalen dan de "normale" client-software voor gewone gebruikers. Voorbeelden:
    - Webservice: `wget http://192.168.56.24/`, `curl https://192.168.56.24/`
    - Samba: `nmblookup server`, `smbclient -Uuser%password //server/share`
    - FTP: `ftp server` of `curl --user user:password ftp://server/path/to/file`
    - DNS: `dig`
    - DHCP: `nmap` heeft [een script](https://nmap.org/nsedoc/scripts/dhcp-discover.html) waarmee je dit kan testen
- Soms is het nuttig om een packet sniffer zoals Tcpdump of Wireshark te gebruiken om de netwerktrafiek tussen clients en de service te bestuderen. Tegenwoordig is veel netwerkverkeer geëncrypteerd, dus de kans bestaat dat dit niet veel nuttige informatie oplevert.

## Troubleshooting tips voor specifieke services en situaties

### BIND

TODO

### VirtualBox netwerkinstellingen

TODO

## TL;DR checklist

Hieronder vind je een overzicht van de belangrijkste zaken die je moet controleren, en de commando's die je daarvoor kan gebruiken.

1. Fysieke en netwerktoegangslaag
    - Controleer of de kabels goed aangesloten zijn
    - Controleer of de kabel nog goed werkt
    - Controleer de LEDs van de switchpoort
    - Is de machine verbonden met de juiste switch?
    - Krijgt de machine een signaal op de kabel? `ip link`
2. Internetlaag
    1. Controleer eerst lokale netwerkinstellingen
        - IP adres (`ip a`)
        - default gateway (`ip r`)
        - DNS (`cat /etc/resolv.conf` of `resolvectl`)
    2. Routering binnen het LAN
        - ping naar de gateway en naar andere gekende hosts op het LAN
        - controleer beschikbaarheid van DNS (`nslookup`, `dig`, `getent ahosts`)
    3. Routering naar het internet
        - ping een gekende host buiten het LAN (let er op dat sommige routers ICMP-verkeer blokkeren en pingen naar buiten in dit geval niet werkt)
3. Transportlaag
    1. Is de service opgestart? `systemctl status`
    2. Luistert de service op de juiste netwerkinterface en poortnummer? `sudo ss -tlnp`
    3. Laat de firewall netwerkverkeer naar de service door? `sudo firewall-cmd --list-all`
4. Applicatielaag
    - Controleer systeemlogs: `sudo journalctl -f -u SERVICE`, `tail -f /var/log/LOG_FILE`
    - Gebruik command-line tools om de service aan te spreken (`curl`, `smbclient`, `dig`, ...)
    - Controleer of de service beschikbaar is over het netwerk (`nmap`, `wireshark`, ...)