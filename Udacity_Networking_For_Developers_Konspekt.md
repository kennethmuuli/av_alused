# Udacity, Networking For Developers Konspekt

## From Ping to HTTP käsud ja sisu selgitus
### `ping -cN N.N.N.N` 
Ping on käsk (kus N = mingi number), millega testida, kas sinu arvuti suudab saata ja vastuvõtta liiklust konkreetselt aadressilt. -cN tähendab, et saata tuleb N (kus N on number, mis sätestab arvu) test sõnumit, siis lõpetada ja tulemused kuvada terminali.

Kui saad vastuseks, et saadeti N sõnumit ja tagasi tuli N sõnumit, 0% sisu kaoga ("packet loss") võib järeldada, et:
  - sinu arvuti on internetiga ühendatud
  - N.N.N.N aadressil olev arvuti töötab
  - sinu interneti teenuse pakkuja (ISP) teab, kuidas saata liiklust antud aadressile 


## Ei olnud võimalik sisestada
- tcpdump -n -c5 -i eth0 port 22
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
