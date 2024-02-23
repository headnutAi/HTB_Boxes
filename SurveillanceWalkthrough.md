# Surveillance

- Headnut | Demian - Mittwoch, 21. 03. 2024

----

#### Nmap/Rustscan
`files/www-data/nmap.txt`

Using Nmap I saw that the remote machine had 3 open ports.
1. 22/tcp - ssh - 3ubuntu0.4
2. 80/tcp - http - nginx 1.18.0 (Ubuntu)
3. 8000/tcp - http - SimpleHTTPServer 0.6

#### Feroxbuster
`files/www-data/feroxbuster-scan.json`

Using Feroxbuster one can find a admin login page under: `http://surveillance.htb/admin` that uses and outdated version of [Craft CMS](https://craftcms.com), version `4.4.14`
One can find the version that craft cms is running on when one inspects the source code on the main page.

#### Craft CMS vulnerability
The version of [Craft CMS](https://craftcms.com) used by the Website is vulnerable to [CVE-2023-41892]("https://www.cvedetails.com/cve/CVE-2023-41892/").

Using a [script](https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce)  That one can find on github you can make a reverse shell to get access to the machine.
- `python3 /opt/craft-cms.py http://surveillance.htb`

#### Privilege Escalation
- linpeas scan under: `files/www-data/linpeas.txt`.
- tree of the website: `files/www-data/tree.txt`

Tho you can not find anything by scanning through linppeas or the tree of the website. So you have to look out for interesting files yourself. One can find a backup file for their DB by looking through the data.
- `/var/www/html/craft/storage/backups/surveillance--2023-10-17-202801--v4.4.14.sql.zip`

You will have to download the file and then extract it to open the file.

Theres a lot of useless Information in this file. But after going through the file a bit more i stumbled on a `INSERT INTO 'users'` sql command. This command inserted a new user into the database named `Matthew B` with his E-Mail address and a hash of his password, likely a `SHA-256` hash. NOICE!!!11!1!!

- email: `admin@surveillance.htb`
- hash: `39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec`

#### Bruteforcing Matthew's password
Using `hash-identifier` one can determine that its a `SHA-256` hash.

To crack the hash i used `hashcat`. After some time i had the password of Matthew: `starcraft122490`. The password only works for SSH access
- `hashcat -m 1400 Matthew.txt /usr/share/wordlists/rockyou.txt`

#### Connection with ssh
We can now connect with the username and password previously found. The user flag can be found under `~/user.txt`.

One can ran `linpeas.sh` once again, under the section `Analyzing Backup Manager Files` one will find a configuration for `ZoneMinder`. After a bit of research I found out `ZoneMinder` had a dashboard which was accessable under 127.0.0.1:2222. I forwarded the port using `ssh -L 2222:127.0.0.1:8080 matthew@surveillance.htb`.

Accessing `127.0.0.1:2222`, we can find a login form for ZoneMinder. But none of the standard credentials or even matthews login works.

Although one can not access the /www one can access the /db in which you can find all the updates made to the software so thats how one can find out the version of ZoneMinder. [ZoneMinder-1.36.32]("https://github.com/ZoneMinder/zoneminder/releases/tag/1.36.32")

#### ZoneMinder
With a [python script](https://github.com/rvizx/CVE-2023-26035) from GitHub One can once again push a reverse shell onto the server.

Using the version and the knowledge of google one can find that you can use the `/usr/bin/zmupdate.pg` file to get root permission.
- `sudo /usr/bin/zmupdate.pg --version=1 --user='$(/bin/bash -i)' --pass=YOUR_ZONEMINDER_PASSWORD`

  As soon as you get root permission you can generate a reverse bash -i shell with which you can get further access especially to the /root directory in which the root flag lies in.

#### Conclusion
It was an Interisting machine to pwn, tho it mainly relied on the missed updates by the creators which are ofc intended. Sadly the machine/shell was very unresponsive/laggy , which it to be expected for a free service but it still was pretty annoying to hack with. So i hope someday [HackTheBox](https://app.hackthebox.com) may improve their machine capabilities. GET PWNED BOZO!! Headnut was here!1!
