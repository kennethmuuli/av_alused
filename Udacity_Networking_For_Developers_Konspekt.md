# Udacity, Networking For Developers Konspekt

Enne Google Cloud Shellis käskude kasutamist sisestada:
```
sudo apt-get update
sudo apt-get install netcat-openbsd tcpdump traceroute mtr
```

## From Ping to HTTP käsud ja sisu selgitus
### `ping -cN N.N.N.N` 
Ping on käsk (kus N = mingi number), millega testida, kas sinu arvuti suudab saata ja vastuvõtta liiklust konkreetselt aadressilt. -cN tähendab, et saata tuleb N (kus N on number, mis sätestab arvu) test sõnumit, siis lõpetada ja tulemused kuvada terminali.

Kui saad vastuseks, et saadeti N sõnumit ja tagasi tuli N sõnumit, 0% sisu kaoga ("packet loss") võib järeldada, et:
  - sinu arvuti on internetiga ühendatud
  - N.N.N.N aadressil olev arvuti töötab
  - sinu interneti teenuse pakkuja (ISP) teab, kuidas saata liiklust antud aadressile 

### `printf 'HEAD / HTTP/1.1\r\nHost: en.wikipedia.org\r\n\r\n' | nc en.wikipedia.org 80`
`printf '...'` on käsk, mis prindib terminali, mis iganes on selle järgi kirjutatud, kuid on veidi targem kui `echo '...'` käsk. Printf lubab näiteks sisestada ka realõppusid, mis echo prindiks terminali lihtsalt üksteise järgi kokku.

`nc ...` viitab netcat tööriistale, mille abil saab manuaalselt suhelda veebiteenustega (välja lugemine või sisse kirjutamine), kasutades selle tarbeks TCP (Transmission Control Protocol) või UDP (User Datagram Protocol). Oluline on antud juhul mõista, et nc ise ei tea HTTP-st mitte midagi. See saadab enda sisendi, antud näites en.wikipedia.org otse etteantud porti ja kuna antud juhul vastaspoolel olev server seda HTTP päringuna loeb, saadab see vastuse tagasi.

`|` pipe (püstkriips) on standard UNIXi süntaks, mille abil saab võtta ühe programmi väljundi ja anda selle sisendiks järgmisele programmile. Antud juhul võtame HTTP päringu ja kasutame seda netcati sisendiks. Antud juhul näitab printfis sisend en.wikipedia.org, millist veebilehte majutaja juurest soovime saada ja teine en.wikipedia.org netcati juures, millise majutajaga peaks nc ühenduma.

### Veebiprotokollide kihid
![image](https://user-images.githubusercontent.com/115208151/195994845-54b90440-afdd-4644-a591-b38fef170921.png)

**Transkript kursuses sisalduvate veebiprotokollide teemal**
*There are several different layer models for talking about network protocols. In this course, we'll be using the IETF model, which looks like this. The top layer is the application layer. Protocol such as htp or SSH are application protocols. Concepts we see at this layer are things like, URL's, passwords, the head command, server headers, web pages. Things that make sense to specific applications, such as web browsers or SSH clans. Application protocols are based on protocols with a transport layer, like TCP and UDP. These provide things like port numbers. Transport layer protocols are based on the internet layer, which is basically the same as the one single protocol IP. It's at this layer, that we start talking about things like IP addresses and routes. And IP in turn, runs on top of different kinds of hardware, like wi-fi or ethernet or DSL lines. Layers above this, don't need to worry about things like, how strong is my wireless signal. Each of these layers depends on the one below it and provides particular guarantees to the layers above it. You can think of them as offering and using api's, separating out specific concerns and making it possible to reuse features. Like both http and ssh have some of the same needs. They need the idea of making a connection to a server and those needs are bundled into the TCP layer. Now this layer model isn't perfect, there are a bunch of network protocols that don't exactly fit into any of these layers. But even with these added to the picture, you can notice that all of these things above depend on IP. An IP can depend on various things underneath it. IP itself, is the only thing on its layer. Some folks refer to this is the narrow waste of the internet protocol stack, because everything above and below, goes through IP.*

###

## Ei olnud võimalik sisestada
**tcpdump -n -c5 -i eth0 port 22**
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
