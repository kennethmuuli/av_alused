# Udacity, Networking For Developers Konspekt

Enne Google Cloud Shellis käskude kasutamist sisestada:
```
sudo apt-get update
sudo apt-get install netcat-openbsd tcpdump traceroute mtr iputils-ping lsof
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
  
## Ei olnud võimalik sisestada
  
> **tcpdump -n -c5 -i eth0 port 22**
  
  ```
  kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ tcpdump -n -c5 -i eth0 port 22
  tcpdump: eth0: You don't have permission to capture on that device
  (socket: Operation not permitted)
  ```

**ping -c3 8.8.8.8**
  
```
kenneth_muuli@cloudshell:~ (my-project-1523600208553)$ ping -c3 8.8.8.8
-bash: ping: command not found
```
  
Lahenduseks: installi ping järgmise käsuga `sudo apt-get install iputils-ping`

**Tupikutest välja tulemine Cloud Shellis**

A la selle käsuga `printf "GET / HTTP/1.1\r\nHost: www.example.com\r\n\r\n" | nc www.example.com 80` jään seisu, kus ei saa uusi käske sisestada.
  
Lahenduseks: ctrl + C

> **Kuidas seada ülesse kohaliku faili teekonda Cloud Shellis `>`?**

If you would like to save the results of an nc command to a file, you can do this with the > shell redirection operator. For instance, this will save the results to the file example.txt:

`printf "GET / HTTP/1.1\r\nHost: www.example.com\r\n\r\n" | nc www.example.com 80 > example.txt`
  
**sudo lsof -i**

`sudo: lsof: command not found`
  
Lahenduseks kasuta enne käsklust: `sudo apt install lsof`
