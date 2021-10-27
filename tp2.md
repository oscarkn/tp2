**Prérequis**


**Changer le nom de la machine**

```
sudo hostname node1.tp2.linux
```
On a maintenant:
```
oscar@node1:~$ 
```
Puis

```
sudo nano /etc/hostname
```
Rentrer "node1.tp2.linux" puis sauvegarder.
Vérification:

```
oscar@node1:~$ oscar@node1:~$ cat /etc/hostname
node1.tp2.linux
```

**Config réseau**


**ping 1.1.1.1 fonctionnel**

```
oscar@node1:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=33.6 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=25.2 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=54 time=30.5 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=54 time=26.8 ms
^C
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 25.153/28.990/33.583/3.278 ms
```
**ping ynov.com fonctionnel**
```
oscar@node1:~$ ping ynov.com
PING ynov.com (92.243.16.143) 56(84) bytes of data.
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=1 ttl=50 time=24.7 ms
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=2 ttl=50 time=25.3 ms
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=3 ttl=50 time=72.3 ms
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=4 ttl=50 time=23.9 ms
^C
--- ynov.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 23.871/36.565/72.333/20.657 ms
```
**ping <IP_VM> fonctionnel, depuis le PC**
```
PS C:\Users\oscar> ping 192.168.79.6

Pinging 192.168.79.6 with 32 bytes of data:
Reply from 192.168.79.6: bytes=32 time<1ms TTL=64
Reply from 192.168.79.6: bytes=32 time<1ms TTL=64
Reply from 192.168.79.6: bytes=32 time<1ms TTL=64
Reply from 192.168.79.6: bytes=32 time<1ms TTL=64

Ping statistics for 192.168.79.6:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

**Partie 1: SSH**

**1) Installation du serveur**

```
oscar@node1:~$ sudo apt install openssh-server
[sudo] password for oscar:
Reading package lists... Done
Building dependency tree
Reading state information... Done
openssh-server is already the newest version (1:8.2p1-4ubuntu0.3).
0 upgraded, 0 newly installed, 0 to remove and 51 not upgraded.
```

**2) Lancement du service SSH**

```
oscar@node1:~$ systemctl start ssh
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to start 'ssh.service'.
Authenticating as: oscar,,, (oscar)
Password:
==== AUTHENTICATION COMPLETE ===
```

**3) Etude du service SSH**

Statut du service:
```
oscar@node1:~$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-10-25 15:55:27 CEST; 59min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 527 (sshd)
      Tasks: 1 (limit: 1105)
     Memory: 8.3M
     CGroup: /system.slice/ssh.service
             └─527 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

oct. 25 15:55:27 node1.tp2.linux systemd[1]: Starting OpenBSD Secure Shell server...
oct. 25 15:55:27 node1.tp2.linux sshd[527]: Server listening on 0.0.0.0 port 22.
oct. 25 15:55:27 node1.tp2.linux sshd[527]: Server listening on :: port 22.
oct. 25 15:55:27 node1.tp2.linux systemd[1]: Started OpenBSD Secure Shell server.
oct. 25 16:34:54 node1.tp2.linux sshd[23873]: Accepted password for oscar from 192.168.79.1 port 63662 ssh2
oct. 25 16:34:54 node1.tp2.linux sshd[23873]: pam_unix(sshd:session): session opened for user oscar by (uid=0)
```
Les processus liés au service ssh:
```
oscar@node1:~$ ps -ef | grep ssh
root         527       1  0 15:55 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
oscar        855     772  0 15:55 ?        00:00:00 /usr/bin/ssh-agent /usr/bin/im-launch startxfce4
root       23873     527  0 16:34 ?        00:00:00 sshd: oscar [priv]
oscar      23955   23873  0 16:34 ?        00:00:00 sshd: oscar@pts/1
oscar      24208   23956  0 16:58 pts/1    00:00:00 grep --color=auto ssh
```

Le port utilisé par le service ssh:

```
oscar@node1:~$ sudo ss -ltnp
LISTEN                  0                       128                                            0.0.0.0:22                                          0.0.0.0:*                      users:(("sshd",pid=527,fd=3))

LISTEN                  0                       128                                               [::]:22                                             [::]:*                      users:(("sshd",pid=527,fd=4))
```
Les logs du service ssh
```
oscar@node1:/$ journalctl -xe -u ssh
-- Support: http://www.ubuntu.com/support
--
-- A start job for unit ssh.service has finished successfully.
--
-- The job identifier is 130.
oct. 19 17:25:09 oscar-VirtualBox sshd[753]: Accepted password for oscar from 192.168.79.1 port 57321 ssh2
oct. 19 17:25:09 oscar-VirtualBox sshd[753]: pam_unix(sshd:session): session opened for user oscar by (uid=0)
oct. 19 17:25:49 oscar-VirtualBox sshd[849]: Accepted password for oscar from 192.168.79.1 port 57324 ssh2
oct. 19 17:25:49 oscar-VirtualBox sshd[849]: pam_unix(sshd:session): session opened for user oscar by (uid=0)
```
Connexion au serveur:
```
PS C:\Users\oscar> ssh oscar@192.168.79.6
oscar@192.168.79.6's password:
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-38-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

51 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Mon Oct 25 16:34:54 2021 from 192.168.79.1
oscar@node1:~$ 
```
**Modification du comportement de service**

```
oscar@node1:~$ oscar@node1:~$ cat /etc/ssh/sshd_c
Port 42069
```
```
oscar@node1:~$ sudo systemctl restart sshd
oscar@node1:~$ sudo ss -ltnp
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port   Process
LISTEN   0        128              0.0.0.0:42069            0.0.0.0:*       users:(("sshd",pid=24459,fd=3))

LISTEN   0        128                 [::]:42069               [::]:*       users:(("sshd",pid=24459,fd=4))
```
Connexion sur le nouveau port choisi:
```
PS C:\Users\oscar> ssh oscar@192.168.79.6 -p 42069
oscar@192.168.79.6's password:
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-38-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

51 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Mon Oct 25 17:26:55 2021 from 192.168.79.1
oscar@node1:~$
```
On doit écrire "-p 42069". Avant ce n'était pas nécessaire de préciser le port car c'est implicit que ssh est dans le port 22.


**Partie 2: FTP**

**1) Installation du serveur**

```
oscar@node1:~$ sudo apt install vsftpd
```

**2) Lancement du service FTP**

```
oscar@node1:~$ systemctl start vsftpd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to start 'vsftpd.service'.
Authenticating as: oscar,,, (oscar)
Password:
==== AUTHENTICATION COMPLETE ===
```
Vérification
```
oscar@node1:~$ systemctl status vsftpd
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-10-27 11:36:09 CEST; 17min ago
```

**3) Etude du service FTP**

Statut du service:

```
oscar@node1:~$ systemctl status vsftpd
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-10-27 11:36:09 CEST; 17min ago
```
Les processus liés au service vsftpd

```
oscar@node1:~$ ps -ef | grep vsftpd
root        3379       1  0 11:36 ?        00:00:00 /usr/sbin/vsftpd /etc/vsftpd.conf
oscar       4114    1324  0 11:55 pts/1    00:00:00 grep --color=auto vsftpd
```
Le port utilisé par le service vsftpd

```
oscar@node1:~$ sudo ss -ltnp
[sudo] password for oscar:
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port   Process
LISTEN   0        32                     *:21                     *:*       users:(("vsftpd",pid=3379,fd=3))
```

Les logs du service vsftpd

```
oscar@node1:~$ journalctl -xe -u vsftpd
-- Logs begin at Tue 2021-10-19 16:29:44 CEST, end at Wed 2021-10-27 14:43:45 CEST. --
oct. 27 11:36:09 node1.tp2.linux systemd[1]: Starting vsftpd FTP server...
-- Subject: A start job for unit vsftpd.service has begun execution
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
--
-- A start job for unit vsftpd.service has begun execution.
--
-- The job identifier is 2841.
```
Connexion au serveur

```
oscar@node1:~$ cat /etc/vsftpd.conf
```
Le # devant "write_enable=YES" doit être enlevé.
```
write_enable=YES
```
Après avoir fait un upload PC > VM :
```
oscar@node1:~$ sudo cat /var/log/vsftpd.log
Wed Oct 27 15:59:04 2021 [pid 4455] [oscar] OK MKDIR: Client "::ffff:192.168.79.1", "/home/oscar/Documents/sendtovm"
Wed Oct 27 15:59:04 2021 [pid 4455] [oscar] OK MKDIR: Client "::ffff:192.168.79.1", "/home/oscar/Documents/sendtovm/sendtopc"
```
Après avoir envoyé un fichier de la VM vers le PC:
```
Wed Oct 27 16:21:28 2021 [pid 5030] CONNECT: Client "::ffff:192.168.79.1"
Wed Oct 27 16:21:28 2021 [pid 5029] [oscar] OK LOGIN: Client "::ffff:192.168.79.1"
```
(ce qu'on voit sur FileZilla)
```
Status:	Retrieving directory listing of "/home/oscar/Documents"...
Status:	Directory listing of "/home/oscar/Documents" successful
```
Vérification que l'upload fonctionne:

```
oscar@node1:~/Documents$ ls
sendtopc  sendtovm
```

**Visualiser les logs**

Ligne de log pour le download

```
Wed Oct 27 16:21:28 2021 [pid 5030] CONNECT: Client "::ffff:192.168.79.1"
Wed Oct 27 16:21:28 2021 [pid 5029] [oscar] OK LOGIN: Client "::ffff:192.168.79.1"
```

Ligne de log pour le upload
```
oscar@node1:~$ sudo cat /var/log/vsftpd.log
Wed Oct 27 15:59:04 2021 [pid 4455] [oscar] OK MKDIR: Client "::ffff:192.168.79.1", "/home/oscar/Documents/sendtovm"
Wed Oct 27 15:59:04 2021 [pid 4455] [oscar] OK MKDIR: Client "::ffff:192.168.79.1", "/home/oscar/Documents/sendtovm/sendtopc"
```

**4) Modification de la configuration du serveur**

**Modifier le comportement du service**

```
oscar@node1:~/Documents$ oscar@node1:~/Documents$ sudo nano /etc/vsftpd.conf
```
On insère
```
listen_port=42070
```
Puis
```
oscar@node1:~/Documents$ systemctl restart vsftpd
```
```
oscar@node1:~/Documents$ sudo ss -ltnp
LISTEN   0        32                     *:42070                  *:*       users:(("vsftpd",pid=5156,fd=3))
```
**Connexion sur le nouveau port choisi**

(sur FileZilla, le port à utiliser est maintenant 42070)

Download:

```
Wed Oct 27 17:31:07 2021 [pid 5268] [oscar] OK LOGIN: Client "::ffff:192.168.79.1"
Wed Oct 27 17:31:58 2021 [pid 5273] CONNECT: Client "::ffff:192.168.79.1"
```

Upload:

```
Wed Oct 27 17:31:58 2021 [pid 5274] [oscar] OK MKDIR: Client "::ffff:192.168.79.1", "/home/oscar/Documents/sendtovm2"
Wed Oct 27 17:31:58 2021 [pid 5274] [oscar] OK MKDIR: Client "::ffff:192.168.79.1", "/home/oscar/Documents/sendtovm2/sendtopc"
```
Vérification:
```
oscar@node1:~/Documents$ ls
sendtopc2  sendtovm  sendtovm2
```