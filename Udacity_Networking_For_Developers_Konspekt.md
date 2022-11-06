# Udacity, Networking For Developers Konspekt

GCS otselink: https://console.cloud.google.com/home/dashboard

Enne Google Cloud Shellis käskude kasutamist sisestada:
```
sudo apt-get update
sudo apt-get install netcat-openbsd tcpdump traceroute mtr iputils-ping lsof -y
```

## From Ping to HTTP käsud ja sisu selgitus

<details>
  <summary>Vaata lähemalt</summary>
  
### `ping -cN N.N.N.N` 

Ping on käsk (kus N = mingi number), millega testida, kas sinu arvuti suudab saata ja vastuvõtta liiklust konkreetselt aadressilt. -cN tähendab, et saata tuleb N (kus N on number, mis sätestab arvu) test sõnumit, siis lõpetada ja tulemused kuvada terminali.
  
Kui saad vastuseks, et saadeti N sõnumit ja tagasi tuli N sõnumit, 0% sisu kaoga ("packet loss") võib järeldada, et:
  - sinu arvuti on internetiga ühendatud
  - N.N.N.N aadressil olev arvuti töötab
  - sinu interneti teenuse pakkuja (ISP) teab, kuidas saata liiklust antud aadressile 

### `printf 'HEAD / HTTP/1.1\r\nHost: en.wikipedia.org\r\n\r\n' | nc en.wikipedia.org 80`

`printf '...'` on käsk, mis prindib terminali, mis iganes on selle järgi kirjutatud, kuid on veidi targem kui `echo '...'` käsk. Printf lubab näiteks sisestada ka realõppusid, mis echo prindiks terminali lihtsalt üksteise järgi kokku.

`\r` viib rea markeri tagasi rea algusesse

`\n` tekitab uue rea
  
`\r\n` on justkui "Enter" klahvi vajutamine

`nc ...` viitab netcat tööriistale, mille abil saab manuaalselt suhelda veebiteenustega (välja lugemine või sisse kirjutamine), kasutades selle tarbeks TCP (Transmission Control Protocol) või UDP (User Datagram Protocol). Oluline on antud juhul mõista, et nc ise ei tea HTTP-st mitte midagi. See saadab enda sisendi, antud näites en.wikipedia.org otse etteantud porti ja kuna antud juhul vastaspoolel olev server seda HTTP päringuna loeb, saadab see vastuse tagasi.

`|` pipe (püstkriips) on standard UNIXi süntaks, mille abil saab võtta ühe programmi väljundi ja anda selle sisendiks järgmisele programmile. Antud juhul võtame HTTP päringu ja kasutame seda netcati sisendiks. Antud juhul näitab printfis sisend en.wikipedia.org, millist veebilehte majutaja juurest soovime saada ja teine en.wikipedia.org netcati juures, millise majutajaga peaks nc ühenduma.

### Veebiprotokollide kihid
<details>
  <summary>Transkript kursuses sisalduvate veebiprotokollide teemal</summary>
  
![image](https://user-images.githubusercontent.com/115208151/195994845-54b90440-afdd-4644-a591-b38fef170921.png)

*There are several different layer models for talking about network protocols. In this course, we'll be using the IETF model, which looks like this. The top layer is the application layer. Protocol such as htp or SSH are application protocols. Concepts we see at this layer are things like, URL's, passwords, the head command, server headers, web pages. Things that make sense to specific applications, such as web browsers or SSH clans. Application protocols are based on protocols with a transport layer, like TCP and UDP. These provide things like port numbers. Transport layer protocols are based on the internet layer, which is basically the same as the one single protocol IP. It's at this layer, that we start talking about things like IP addresses and routes. And IP in turn, runs on top of different kinds of hardware, like wi-fi or ethernet or DSL lines. Layers above this, don't need to worry about things like, how strong is my wireless signal. Each of these layers depends on the one below it and provides particular guarantees to the layers above it. You can think of them as offering and using api's, separating out specific concerns and making it possible to reuse features. Like both http and ssh have some of the same needs. They need the idea of making a connection to a server and those needs are bundled into the TCP layer. Now this layer model isn't perfect, there are a bunch of network protocols that don't exactly fit into any of these layers. But even with these added to the picture, you can notice that all of these things above depend on IP. An IP can depend on various things underneath it. IP itself, is the only thing on its layer. Some folks refer to this is the narrow waste of the internet protocol stack, because everything above and below, goes through IP.*

</details>
  
### TCP Server 

`nc -l NNNN` Võimaldab kuulata porti numbriga NNNN (käitub serverina). Porti võib võrrelda telefoni numbriga, igal teenusel on enda unikaalne telefoni number, näiteks 80 = HTTP ja 20 = SSH. Kui porti kuulata tähendab see justkui vastuvõtja on olemas. Kui porti ei kuulata saaksime vastuseks midagi sarnast nagu "see telefon ei ole levialas". *Madalaim vabalt, e. ilma `sudo` lisata kuulatav port on 1024 (1023 ja alla on reserveeritud programmidele, mille käivitab süsteemi "superuser" või ROOT UNIXi süsteemidel), kõrgeim port 65535

`nc localhost NNNN` (kohalik ühendus) või `nc N.N.N.N NNNN` (kaugühendus) ühendub (käitub kliendina) nimetatud aadressil (N.N.N.N) oleva masina pordiga (NNNN). Oluline mõista, et ühenduda saab ainult klient.

`^D` (Control D) lõpetab sessiooni, sessiooni saavad lõpetada mõlemad osapooled.

<details>
  <summary>nc manuaali väljavõte: "Client/Server Model"</summary>

*"It is quite simple to build a very basic client/server model using nc.  On one console, start nc listening on a specific port for a connection. For example:*
`nc -l 1234`

*nc is now listening on port 1234 for a connection.  On a second console (or a second machine), connect to the machine and port being listened on:*
`nc 127.0.0.1 1234`

*There should now be a connection between the ports. Anything typed at the second console will be concatenated to the first, and vice-versa.  After the connection has been set up, nc does not really care which side is being used as a ‘server’ and which side is being used as a ‘client’.  The connection may be terminated using an EOF (‘^D’).*

*There is no -c or -e option in this netcat, but you still can execute a command after connection being established by redirecting file descriptors. Be cautious here because opening a port and let anyone connected execute arbitrary command on your site is DANGEROUS. If you really need to do this, here is an example:*
     
*On ‘server’ side:*

```
rm -f /tmp/f; mkfifo /tmp/f
cat /tmp/f | /bin/sh -i 2>&1 | nc -l 127.0.0.1 1234 > /tmp/f
```

*On ‘client’ side:*

```
nc host.example.com 1234
(shell prompt from host.example.com)
```
*By doing this, you create a fifo at /tmp/f and make nc listen at port 1234 of address 127.0.0.1 on ‘server’ side, when a ‘client’ es‐tablishes a connection successfully to that port, /bin/sh gets executed on ‘server’ side and the shell prompt is given to ‘client’ side.*

*When connection is terminated, nc quits as well. Use -k if you want it keep listening, but if the command quits this option won't restart it or keep nc running. Also don't forget to remove the file descriptor once you don't need it anymore:*
`rm -f /tmp/f`"

</details>

### HTTP päised ja tegumid

https://learn.udacity.com/courses/ud256/lessons/5a6ddf10-29e2-4d97-a119-507e5a63273f/concepts/5c5acb98-0dbf-45f5-9441-4a8be325e10c

HTTP päringute nimekiri:
https://en.wikipedia.org/wiki/List_of_HTTP_header_fields

<details>
  <summary>nc manuaali väljavõte: "Talking to Servers"</summary>
  
It is sometimes useful to talk to servers “by hand” rather than through a user interface.  It can aid in troubleshooting, when it might be necessary to verify what data a server is sending in response to commands issued by the client.  For example, to retrieve the home page of a web site:
  
```
printf "GET / HTTP/1.0\r\n\r\n" | nc host.example.com 80
```

Note that this also displays the headers sent by the web server.  They can be filtered, using a tool such as sed(1), if necessary.

More complicated examples can be built up when the user knows the format of requests required by the server.  As another example, an email may be submitted to an SMTP server using:
  
```  
nc [-C] localhost 25 << EOF
           HELO host.example.com
           MAIL FROM:<user@host.example.com>
           RCPT TO:<user2@host.example.com>
           DATA
           Body of email.
           .
           QUIT
           EOF
```
  
</details>

**HTTP verbid HEAD, GET, POST**
- HEAD on HTTP tegum, mis palub serveril saata ainult ressursi päised selle asemel, et saata kogu andmestiku.
- GET on HTTP tegum, mis palub serveril saata kogu ressursi andmestiku.
> POST on HTTP tegub, mis võimaldab luua serverisse uusi ressursse?
  
### `printf 'HTTP/1.1 302 Moved\r\nLocation: https://www.eff.org/' | nc -l 2345`

`302 Moved\r\nLocation:` määrame ümber suunamise https://www.eff.org lehele.
  
> Mida antud kontekstis see 302 tähendab?
  
</details>
  
## Names and Addresses käsud ja sisu selgitus

<details>
  <summary>Vaata lähemalt</summary>

### `host example.com`
  
Võimaldab vaadata DNSi sissekandeid inimloetavas vormis.
  
`host -t a example.com` pärib konkreetselt IPv4 aadressi. *Märkus, aka. "A record"*

<details>
  <summary>host manuaali väljavõte: "Description & Options"</summary>  

DESCRIPTION
Host is a simple utility for performing DNS lookups. It is normally used to convert names to IP addresses and vice  versa.  When  no arguments or options are given, host prints a short summary of its command-line arguments and options.

Name  is the domain name that is to be looked up. It can also be a dotted-decimal IPv4 address or a colon-delimited IPv6 address, in which case host by default performs a reverse lookup for that address.  server is an optional argument which is either the  name  or IP address of the name server that host should query instead of the server or servers listed in /etc/resolv.conf.
  
</details>

### `dig example.com`
  
Erinevalt `host` käsust, tagastab antud käsk DNSi sissekanded lähemale nende algsel kujul. Seetähendab, sarnaselt nagu neid ootaks mõni skript ja lähemal kujule nagu need on tegelikult salvestatud. Antud käsk näitab meile ka täiendavat informatsiooni nagu, milline server meie päringule vastas ja millal päringule vastati.
  
### DNS sissekande tüübid
  
![image](https://user-images.githubusercontent.com/115208151/196042206-a5b88b0d-e6bd-495d-aabe-5939cbedf489.png)

<details>
  <summary>DNS type definitions</summary>  

- What is a DNS A record? The "A" stands for "address" and this is the most fundamental type of DNS record: it indicates the IP address of a given domain. For example, if you pull the DNS records of cloudflare.com, the A record currently returns an IP address of: 104.17. 210.9.
- A Canonical Name or CNAME record is a type of DNS record that maps an alias name to a true or canonical domain name. CNAME records are typically used to map a subdomain such as www or mail to the domain hosting that subdomain's content.
- An AAAA record maps a domain name to the IP address (Version 6) of the computer hosting the domain. An AAAA record is used to find the IP address of a computer connected to the internet from a name.
- NS = An NS record (or nameserver record) is a DNS record that contains the name of the authoritative name server within a domain or DNS zone. When a client queries for an IP address, it can find the IP address of their intended destination from an NS record via a DNS lookup.
  
</details>
  
</details>

## Addressing and networks käsud ja sisu selgitus

<details>
  <summary>Vaata lähemalt</summary> 
  
### Network blocks

Network block näitab, mitu aadressi on antud võrgus kasutusel konkreetse masinani edasi suunamiseks (host aadressiks). Seda märgitakse tavaliselt IP aadressi järgi näiteks nii: `123.123.123.123 /24` kus /24 näitab võrgu eesliite (prefixi) pikkust. See tähendab, et 24 bitti kasutatakse eesliidesena (network part) ja ülejäänud 8 bitti kasutatakse kohalikuks suunamiseks (host part). Tuleta meelde bittide teisendamist kümnend süsteemi. 8 bitti ei tähenda loomulikult, et host aadresse on vaid 8 tükki 2 astmes 8 aadressi ehk 256. Praktikas aga on nendest kasutatavad 253 või vähem aadressi, kuna 3 reeglina on esimene ja viimane aadress tühjad ning esimene vaba aadress ruuteri oma.
  
Network blocki pikkust on võimalik tuvastada ka selle maski järgi (subnet mask), näiteks 255.255.255.0 (hexina ff ff ff 0) viitab 24 bitisele network blockile. 16 bitine network block aga kannaks subnet maski 255.255.0.0.
  
### `ip addr show`
  
Käsuga on võimalik näha masina interface, millel on tegelikult küljes IP (masina endal otseselt IP-d ei ole). inet real on näha IPv4 aadresse ja nende network blockide suuruseid.

Näidis tulemus:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:3b:80:76:50 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever  
```

Loopback interface on nende seas eriline. Sellel on pea alati aadress 127.0.0.1 (local host) ja see võimaldab programmidel samal interfacel rääkida teiste programmidega.

### `ifconfig | less`
  
Käsk annab sarnase tulemuse, mis `ip addr show` kuid toob esile palju rohkem infot, s.t. kõik interfaceid, millest enamus ei ole seotud interneti liiklusega, kuid käsk on kasulik näiteks printeri leidmiseks.
  
## default gateway

`ip route show default` (Linux) ja `netstat -nr` (Linux, Mac, Unix) on käsud, millega on võimalik leida enda gateway (ruuteri) default aadressi.
  
</details>

## Protocol Layers
  
<details>
  <summary>Vaata lähemalt</summary> 
  
### Protocol Layers breakdown and structure
  
<details>
  <summary>Breakdown & Structure</summary> 
  
![image](https://user-images.githubusercontent.com/115208151/199561047-84452f51-1c11-461c-8475-89ef9d229a0b.png)
  
![image](https://user-images.githubusercontent.com/115208151/199561425-d729a286-24d9-4152-bb8b-e0daf34f336e.png)

</details>  

### `sudo tcpdump -n host 8.8.8.8`

`tcpdump` kuvatud info ülesehitus ühe pingi kohta:

```  
17:40:40.600401 IP 172.17.0.4 > 8.8.8.8: ICMP echo request, id 16682, seq 1, length 64
17:40:40.601734 IP 8.8.8.8 > 172.17.0.4: ICMP echo reply, id 16682, seq 1, length 64
```

*Ühe pingi kohta kuvab tcpdump välja kaks rida, päring ja vastus. Igal real on näha (muuseas) päring, IP millelt päring tehakse, IP kuhu päring läheb (või kust vastus tuleb ja kuhu) ning viimane element näitab paketi pikkust, 64 bitti, mis vastab pingi paketi suurusele 64 bitti.*

Samal ajal teises Google Cloud Shell instantsis jooksev `ping 8.8.8.8` käsk

```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=1.37 ms
```

**MÄRKUSED:**

`port 53` näitab väljuvaid DNS päringuid.

`port 80` näitab pakette, mida minu masin kasutab veebiserveritega suhtlemiseks
  
### tcpdump ülesanne

Käsk: `printf 'HEAD / HTTP/1.1\r\nHost: example.net\r\n\r\n' | nc example.net 80`

```
kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ sudo tcpdump -n host example.net
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
18:00:24.499751 IP 172.17.0.4.35124 > 93.184.216.34.80: Flags [S], seq 2685880152, win 64240, options [mss 1460,sackOK,TS val 2054948752 ecr 0,nop,wscale 7], length 0
18:00:24.579154 IP 93.184.216.34.80 > 172.17.0.4.35124: Flags [S.], seq 750517273, ack 2685880153, win 65535, options [mss 1460,sackOK,TS val 1460704456 ecr 2054948752,nop,wscale 9], length 0
18:00:24.579190 IP 172.17.0.4.35124 > 93.184.216.34.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 2054948831 ecr 1460704456], length 0
18:00:24.579270 IP 172.17.0.4.35124 > 93.184.216.34.80: Flags [P.], seq 1:37, ack 1, win 502, options [nop,nop,TS val 2054948831 ecr 1460704456], length 36: HTTP
18:00:24.657784 IP 93.184.216.34.80 > 172.17.0.4.35124: Flags [.], ack 37, win 128, options [nop,nop,TS val 1460704535 ecr 2054948831], length 0
18:00:24.657787 IP 93.184.216.34.80 > 172.17.0.4.35124: Flags [P.], seq 1:517, ack 37, win 128, options [nop,nop,TS val 1460704535 ecr 2054948831], length 516: HTTP: HTTP/1.0 501 Not Implemented
18:00:24.657827 IP 172.17.0.4.35124 > 93.184.216.34.80: Flags [.], ack 517, win 498, options [nop,nop,TS val 2054948910 ecr 1460704535], length 0
18:00:24.657789 IP 93.184.216.34.80 > 172.17.0.4.35124: Flags [F.], seq 517, ack 37, win 128, options [nop,nop,TS val 1460704535 ecr 2054948831], length 0
18:00:24.657935 IP 172.17.0.4.35124 > 93.184.216.34.80: Flags [F.], seq 37, ack 518, win 501, options [nop,nop,TS val 2054948910 ecr 1460704535], length 0
18:00:24.736390 IP 93.184.216.34.80 > 172.17.0.4.35124: Flags [.], ack 38, win 128, options [nop,nop,TS val 1460704614 ecr 2054948910], length 0
18:01:24.877731 IP 172.17.0.4.40890 > 93.184.216.34.80: Flags [S], seq 109788949, win 64240, options [mss 1460,sackOK,TS val 2055009130 ecr 0,nop,wscale 7], length 0
18:01:24.956367 IP 93.184.216.34.80 > 172.17.0.4.40890: Flags [S.], seq 1209223184, ack 109788950, win 65535, options [mss 1460,sackOK,TS val 1855821290 ecr 2055009130,nop,wscale 9], length 0
18:01:24.956415 IP 172.17.0.4.40890 > 93.184.216.34.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 2055009208 ecr 1855821290], length 0
18:01:24.956535 IP 172.17.0.4.40890 > 93.184.216.34.80: Flags [P.], seq 1:39, ack 1, win 502, options [nop,nop,TS val 2055009208 ecr 1855821290], length 38: HTTP: HEAD / HTTP/1.1
18:01:25.034494 IP 93.184.216.34.80 > 172.17.0.4.40890: Flags [.], ack 39, win 128, options [nop,nop,TS val 1855821368 ecr 2055009208], length 0
18:01:25.035005 IP 93.184.216.34.80 > 172.17.0.4.40890: Flags [P.], seq 1:335, ack 39, win 128, options [nop,nop,TS val 1855821369 ecr 2055009208], length 334: HTTP: HTTP/1.1 200 OK
18:01:25.035028 IP 172.17.0.4.40890 > 93.184.216.34.80: Flags [.], ack 335, win 500, options [nop,nop,TS val 2055009287 ecr 1855821369], length 0
18:01:33.017308 IP 172.17.0.4.40890 > 93.184.216.34.80: Flags [F.], seq 39, ack 335, win 501, options [nop,nop,TS val 2055017269 ecr 1855821369], length 0
18:01:33.095850 IP 93.184.216.34.80 > 172.17.0.4.40890: Flags [F.], seq 335, ack 40, win 128, options [nop,nop,TS val 1855829430 ecr 2055017269], length 0
18:01:33.095876 IP 172.17.0.4.40890 > 93.184.216.34.80: Flags [.], ack 336, win 501, options [nop,nop,TS val 2055017348 ecr 1855829430], length 0  
```

### Network protokollide töö illustreerimine

<details>
  <summary>Sequence diagram example</summary> 
  
![image](https://user-images.githubusercontent.com/115208151/199568262-261d7ab9-d5d2-48f6-807c-9a34760365e2.png)
  
</details>

### TCP Flags
  
**The six basic TCP flags**

What is a flag?
In low-level computer languages, a flag is a Boolean value — a true or false value — that is stored in memory as a single bit. If a flag bit is 1, we say the flag is set. If the flag bit is 0, the flag is cleared (or unset).

Usually, flags come in groups, each of which can be set or cleared.

The six basic TCP flags
The original TCP packet format has six flags. Two more optional flags have since been standardized, but they are much less important to the basic functioning of TCP. For each packet, tcpdump will show you which flags are set on that packet.

- SYN (synchronize) [S] — This packet is opening a new TCP session and contains a new initial sequence number.
- FIN (finish) [F] — This packet is used to close a TCP session normally. The sender is saying that they are finished sending, but they can still receive data from the other endpoint.
- PSH (push) [P] — This packet is the end of a chunk of application data, such as an HTTP request.
- RST (reset) [R] — This packet is a TCP error message; the sender has a problem and wants to reset (abandon) the session.
- ACK (acknowledge) [.] — This packet acknowledges that its sender has received data from the other endpoint. Almost every packet except the first SYN will have the ACK flag set.
- URG (urgent) [U] — This packet contains data that needs to be delivered to the application out-of-order. Not used in HTTP or most other current applications.

<details>
  <summary>Näita veel infot</summary> 

**Three-way handshake**
The first packet sent to initiate a TCP session always has the SYN flag set. This initial SYN packet is what a client sends to a server to start opening a TCP connection. This is the first packet you see in the sample tcpdump data, with Flags [S]. This packet also contains a new, randomized sequence number (seq in tcpdump output).

If the server accepts the connection, it sends a packet back that has the SYN and ACK flags, and acknowledges the initial SYN. This is the second packet in the sample data, with Flags [S.]. This contains a different initial sequence number.

(If the server doesn't want to accept the connection, it may not send anything at all. Or it may send a packet with the RST flag.)

Finally, the client acknowledges receiving the SYN|ACK packet by sending an ACK packet of its own.

This exchange of three packets is usually called the TCP three-way handshake. In addition to sequence numbers, the two endpoints also exchange other information used to set up the connection.

**Four-way teardown**
  
When either endpoint is done sending data into the connection, it can send a FIN packet to indicate that it is finished. The other endpoint will send an ACK to indicate that it has received the FIN.

In the example HTTP data, the client sends its FIN first, as soon as it is done sending the HTTP request. This is the first packet containing Flags [F.].

Eventually the other endpoint will be done sending as well, and will send a FIN of its own. Then the first endpoint will send an ACK.

**In between**
  
In a long-running connection, there will be many packets exchanged back and forth. Some of them will contain application data; others may be only acknowledgments with no data (length 0). However, all TCP packets in a connection except the initial SYN will contain an acknowledgment of all the data that the sender has received so far. Therefore, they will all have the ACK flag set. (This is why tcpdump depicts the ACK flag with just a dot: it's really common.)

ICMP and UDP don't have TCP flags
If you look at tcpdump data for pings or basic DNS lookups, you will not see flags. This is because ping uses ICMP, and basic DNS lookups use UDP. These protocols do not have TCP flags or sequence numbers.

</details>

</details>
  
## Big Networks
  
<details>
  <summary>Vaata lähemalt</summary>
  
### 'traceroute' ja 'mtr'
  
'traceroute example.com' käsk näitab "hüppeid" sinu seadmest kuni päritud aadressini, s.t. millistelt IP-delt/ruuteritelt päring läbi läheb enne kui see jõuab sinu soovitud IP-ni (ja tagasi tuleb).
  
'mtr' käsk näitab sisuliselt sama infot, mida 'traceroute' kuid täpsemalt, kuna see jälgib teekonda elavalt, s.t. "lives" ja teekonda tehakse läbi korduvalt. Käsk võib välja tuua erinevaid teekondasid sihini.
  
## bandwidth vs latency

- bandwidth näitab, kui palju andmeid saab konkreetset kanalit pidi mingis ajaühikus liikuda. Ühik: bits/second
- latency näitab, kui kaua kulub aega andmete ühest kohast jõudmiseks teise.
- bandwidth delay product
  
</details>
  
## Ei olnud võimalik sisestada
  
**MÄRKUSEKS: SISSE KIRJUTAMISEL JÄÄDVUSTA ALATI KA KOHT, KUS PROBLEEM TEKKIS.**
  
> **erinevad default gatewayd**
  
`ip route show default` / `netstat -nr` annavad erinevaid tulemusi. Vaata alla poole:

```
kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ ip route show default
default via 172.17.0.1 dev eth0
kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG        0 0          0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 eth0
172.18.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
```  
  
> **tcpdump -n -c5 -i eth0 port 22**
  
  ```
  kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ tcpdump -n -c5 -i eth0 port 22
  tcpdump: eth0: You don't have permission to capture on that device
  (socket: Operation not permitted)
  ```
Lahenduseks: 
  1) lisa käsule `sudo` -> `tcpdump -n -c5 -i eth0 port 22`
  2) Käsu tulemuse nägemiseks algata teine paralleelne session ja tee ssh päring
  
**PS! Uuri välja, kus seal kursusel sul see probleem tekkis täpselt.**

> **Kuidas seada ülesse kohaliku faili teekonda Cloud Shellis `>`?**

If you would like to save the results of an nc command to a file, you can do this with the > shell redirection operator. For instance, this will save the results to the file example.txt:

`printf "GET / HTTP/1.1\r\nHost: www.example.com\r\n\r\n" | nc www.example.com 80 > example.txt`
  
<details>
  <summary>"Lahendatud"</summary>
  
**sudo lsof -i**

`sudo: lsof: command not found`
  
Lahenduseks kasuta enne käsklust: `sudo apt install lsof`

**Tupikutest välja tulemine Cloud Shellis**

A la selle käsuga `printf "GET / HTTP/1.1\r\nHost: www.example.com\r\n\r\n" | nc www.example.com 80` jään seisu, kus ei saa uusi käske sisestada.
  
Lahenduseks: ctrl + C
  
**ping -c3 8.8.8.8**
  
```
kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ ping -c3 8.8.8.8
-bash: ping: command not found
```
Lahenduseks: installi ping järgmise käsuga `sudo apt-get install iputils-ping`

</details>

## Muud kasulikud käsud

<details>
  <summary>"Vaata lähemalt"</summary>
  
### `pwd`
pwd stands for Print Working Directory. It prints the path of the working directory, starting from the root.

This command has two flags.
```
pwd -L: Prints the symbolic path.
pwd -P: Prints the actual path.
```

</details>
