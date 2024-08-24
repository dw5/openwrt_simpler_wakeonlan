# WOL Frontend For GL-INET edition of OpenWRT

You can use exports from Wake on Lan app  
https://github.com/Florianisme/WakeOnLan  
https://apt.izzysoft.de/fdroid/index/apk/de.florianisme.wakeonlan  

# Why?

Needing to use LU-CI requires need to sign in, then use big pile list of all devices, including wifi ones (tldr: the ones I don't need),  
also Luci WOL listing tends to be confusing (sometime you see hostname, sometimes tailscale hostname, sometimes tailscale IP, sometimes ipv6 fe80:: junk), plus need to speficially use br-lan, so this frontend is just cleaner and just has what you need

# Install

```
add wol.php to /www

opkg install php7 php7-fpm php7-mod-sockets php7-mod-json

opkg install shadow-useradd
useradd -r -s /bin/ash usrphpetherwake

"/etc/sudoers"
usrphpetherwake ALL=(ALL) NOPASSWD: /usr/bin/etherwake

"/etc/php7-fpm.d/www.conf"
nobody -> usrphpetherwake

"/etc/nginx/conf.d/gl.conf"
    location /wol {
        # Internally rewrite /wol to /wol.php
        rewrite ^/wol$ /wol.php last;

        # Process the PHP script using PHP-FPM
        include fastcgi_params;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php7-fpm.sock;  # Update the PHP version/socket as needed
        fastcgi_index wol.php;
        fastcgi_param SCRIPT_FILENAME $document_root/wol.php;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

sudo nginx -t

sudo /etc/init.d/nginx restart
/etc/init.d/php7-fpm restart
```