# Surveillance

- Luca Leukert - Mittwoch, 21. 03. 2024

----

#### Nmap
`files/www-data/nmap.txt`

Using Nmap I saw that the remote machine had 3 open ports.
1. 22/tcp - ssh - 3ubuntu0.4
2. 80/tcp - http - nginx 1.18.0 (Ubuntu)
3. 8000/tcp - http - SimpleHTTPServer 0.6

#### Feroxbuster
`files/www-data/feroxbuster-scan.json`

Feroxbuster found a login page under `http://surveillance.htb/admin` that uses and outdated version of [Craft CMS](https://craftcms.com), version `4.4.14`

#### Craft CMS
The version of [Craft CMS](https://craftcms.com) used by the Website is vulnerable to [CVE-2023-41892]("https://www.cvedetails.com/cve/CVE-2023-41892/").

Using a [script](https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce) I found on GitHub i was able too open a reverse shell to the target machine.
- `python3 /opt/craft-cms.py http://surveillance.htb`

#### Privilege Escalation
- linpeas scan under: `files/www-data/linpeas.txt`.
- tree of the website: `files/www-data/tree.txt`

I did not find anything right way that I could use to priv esc so I did some digging a around by hand. I found a backup files for their database:
- `/var/www/html/craft/storage/backups/surveillance--2023-10-17-202801--v4.4.14.sql.zip`

This file include a lot of irrelevant information. After a file i stumbled on a `INSERT INTO 'users'` sql command. This command inserted a new user into the database named `Matthew B` with his E-Mail address and a hash of his password, likely a `SHA-256` hash. Bingo!!!.

- email: `admin@surveillance.htb`
- hash: `39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec`

#### Bruteforcing Matthew's password
Using `hash-identifier` i could determine that the hash was indeed a `SHA-256` hash.

To crack the hash i used `hashcat`. After about 2 minutes of cracking i had the password of Matthew: `starcraft122490`. The password only works for SSH access
- `hashcat -m 1400 Matthew.txt /usr/share/wordlists/rockyou.txt`

#### This place feels familiar but somehow different
Now we can connect to the box using ssh on a new nice and stable connection. The user flag can be found under `~/user.txt`.

I ran `linpeas.sh` once again, under the section `Analyzing Backup Manager Files` i found a configuration for `ZoneMinder` a software for video surveillance. After a bit of research I found out `ZoneMinder` had a dashboard which was accessable under 127.0.0.1:2222. I forwarded the port using `ssh -L 2222:127.0.0.1:8080 matthew@surveillance.htb`.

Accessing `127.0.0.1:2222`, we can find a login form for ZoneMinder. I tried some common credentials and Matthew's password but their are all wrong.

Using a script that prints all files containing the string `version` i was able too find the version of ZoneMinder. [ZoneMinder-1.36.32]("https://github.com/ZoneMinder/zoneminder/releases/tag/1.36.32")

#### ZoneMinder
With a [python script](https://github.com/rvizx/CVE-2023-26035) from GitHub I was able too once again open a reverse shell now to the user ZoneMinder.

Using the version and the knowlage of google i found that you can use the `/usr/bin/zmupdate.pg` file to get root permission.
- `sudo /usr/bin/zmupdate.pg --version=1 --user='$(/bin/bash -i)' --pass=YOUR_ZONEMINDER_PASSWORD`

#### Conclusion
After having pwned the machine I can say it was a very interesting adventure. Sadly the machine was very unresponsive, which it to be expected for a free service but still their is quite some headroom left for [HackTheBox](https://app.hackthebox.com) to improve. GET PWNED BOZO!!
