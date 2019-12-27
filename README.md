#### Creating Bridge 
- By default we can only ping the private IP of the containers. Inorder to connect the containers via hostname they should be in the same bridge.

````
[root@ip-172-31-14-145 ~]# docker network create --driver=bridge wpnet
a9d38462b35a2d9feef5734bec81d41da8a6e1205bb3e3ddf8f7f322af5d31b5
[root@ip-172-31-14-145 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3a7ef291ff94        bridge              bridge              local
25da814ba41b        host                host                local
036f93dbcb61        none                null                local
a9d38462b35a        wpnet               bridge              local


[root@ip-172-31-14-145 ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
br-a9d38462b35a         8000.0242146f45de       no
docker0         8000.0242da725020       no

[root@ip-172-31-14-145 ~]# ifconfig 
br-a9d38462b35a: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:14:6f:45:de  txqueuelen 0  (Ethernet)
        RX packets 66145  bytes 94372940 (90.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3826  bytes 258638 (252.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:da:72:50:20  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.14.145  netmask 255.255.240.0  broadcast 172.31.15.255
        inet6 fe80::83f:eeff:fe45:46a2  prefixlen 64  scopeid 0x20<link>
        ether 0a:3f:ee:45:46:a2  txqueuelen 1000  (Ethernet)
        RX packets 66145  bytes 94372940 (90.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3835  bytes 260160 (254.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8  bytes 648 (648.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
````

#### Launching Containers in the Bridge
````
[root@ip-172-31-14-145 ~]# docker run -d -it --name c1 --network wpnet alpine:3.8
Unable to find image 'alpine:3.8' locally
3.8: Pulling from library/alpine
c87736221ed0: Pull complete 
Digest: sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9
Status: Downloaded newer image for alpine:3.8
6dfbfcb5af855a37c8fa77a224f8daa5a39c089b09bfa8e5935815288ac8694e


[root@ip-172-31-14-145 ~]# docker run -d -it --name c2 --network wpnet alpine:3.8
9e49b6aab960f10494e50e241a2b6d1e9f8efafd6783e908ea6eab1a88b9ef34


[root@ip-172-31-14-145 ~]# docker network inspect wpnet
[
    {
        "Name": "wpnet",
        "Id": "a9d38462b35a2d9feef5734bec81d41da8a6e1205bb3e3ddf8f7f322af5d31b5",
        "Created": "2019-12-27T08:53:20.782588688Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "6dfbfcb5af855a37c8fa77a224f8daa5a39c089b09bfa8e5935815288ac8694e": {
                "Name": "c1",
                "EndpointID": "815e19e5c092876adece32fd22999b6d96385269b511a27382bc6e58f01ecfb0",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "9e49b6aab960f10494e50e241a2b6d1e9f8efafd6783e908ea6eab1a88b9ef34": {
                "Name": "c2",
                "EndpointID": "856bd45cc30e100ab917cfb0e020421a650a5bc8574d42ab1b94732c1dbbb8f3",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
````

#### Checking Connectivity

````
[root@ip-172-31-14-145 ~]# docker exec c1 ping -c 5 c2
PING c2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=255 time=0.092 ms
64 bytes from 172.18.0.3: seq=1 ttl=255 time=0.075 ms
64 bytes from 172.18.0.3: seq=2 ttl=255 time=0.075 ms
64 bytes from 172.18.0.3: seq=3 ttl=255 time=0.086 ms
64 bytes from 172.18.0.3: seq=4 ttl=255 time=0.073 ms

--- c2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.073/0.080/0.092 ms


[root@ip-172-31-14-145 ~]# docker exec c2 ping -c 5 c1
PING c1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=255 time=0.065 ms
64 bytes from 172.18.0.2: seq=1 ttl=255 time=0.076 ms
64 bytes from 172.18.0.2: seq=2 ttl=255 time=0.075 ms
64 bytes from 172.18.0.2: seq=3 ttl=255 time=0.078 ms
64 bytes from 172.18.0.2: seq=4 ttl=255 time=0.073 ms

--- c1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.065/0.073/0.078 ms
````

#### Creating Storage for Wordpress & Mysql
````
[root@ip-172-31-14-145 ec2-user]# mkdir mysql-data
[root@ip-172-31-14-145 ec2-user]# mkdir wordpress-data

[root@ip-172-31-14-145 ec2-user]# docker run -d -it --name database -v $(pwd)/mysql-data/:/var/lib/mysql --network wpnet --restart always -e MYSQL_ROOT_PASSWORD=mysql456 -e MYSQL_DATABASE=db -e MYSQL_USER=dbuser -e MYSQL_PASSWORD=user456 mysql:5.6
Unable to find image 'mysql:5.6' locally
5.6: Pulling from library/mysql
d599a449871e: Pull complete 
f287049d3170: Pull complete 
08947732a1b0: Pull complete 
96f3056887f2: Pull complete 
871f7f65f017: Pull complete 
111ea1dd4e23: Pull complete 
2dcf2d87da45: Pull complete 
648aa2667757: Pull complete 
418a18378dc0: Pull complete 
02a64522fded: Pull complete 
577b15a8d700: Pull complete 
Digest: sha256:5345afaaf1712e60bbc4d9ef32cc62acf41e4160584142f8d73115f16ad94af4
Status: Downloaded newer image for mysql:5.6
083fd7d125de6cc0d5600c2b90f977b43469fad49af16bcb468627c10ba0b9ae


[root@ip-172-31-14-145 ec2-user]# docker run -d -v $(pwd)/wordpress-data:/var/www/html --name content --network wpnet --restart always -p 80:80 -e WORDPRESS_DB_HOST=database -e WORDPRESS_DB_USER=dbuser -e WORDPRESS_DB_PASSWORD=user456 -e WORDPRESS_DB_NAME=db wordpress:latest
Unable to find image 'wordpress:latest' locally
latest: Pulling from library/wordpress
000eee12ec04: Pull complete 
8ae4f9fcfeea: Pull complete 
60f22fbbd07a: Pull complete 
ccc7a63ad75f: Pull complete 
a2427b8dd6e7: Pull complete 
91cac3b30184: Pull complete 
d6e40015fc10: Pull complete 
bcf656488df0: Pull complete 
9e0f1c96327f: Pull complete 
09286920562e: Pull complete 
7882c813736f: Pull complete 
39600479e1f6: Pull complete 
3fe12f16ff45: Pull complete 
5a0cfb059343: Pull complete 
9173e389a948: Pull complete 
a756278f2e3c: Pull complete 
327c2afd9e2d: Pull complete 
c7aa18fe86ab: Pull complete 
0de3fb734f04: Pull complete 
c065608ab636: Pull complete 
f3050ad9b9e6: Pull complete 
Digest: sha256:a56b186fa6f69ac8cf64edec5cc02daab6f97948e8af01a0d62a8a46b15b5be2
Status: Downloaded newer image for wordpress:latest
a2dc2547ffe59a7f327749f13f04c0577ec84d35b651f59cd40b011104322538
````

#### Checking inside the containers to see the data contents 
````
[root@ip-172-31-14-145 ec2-user]# docker exec -it content bash
root@a2dc2547ffe5:/var/www/html# ls
index.php    wp-activate.php     wp-comments-post.php  wp-content   wp-links-opml.php  wp-mail.php      wp-trackback.php
license.txt  wp-admin            wp-config-sample.php  wp-cron.php  wp-load.php        wp-settings.php  xmlrpc.php
readme.html  wp-blog-header.php  wp-config.php         wp-includes  wp-login.php       wp-signup.php

root@a2dc2547ffe5:/var/www/html# grep -i database wp-config.php
 * * Database table prefix
/** The name of the database for WordPress */
/** MySQL database username */
/** MySQL database password */
define( 'DB_HOST', 'database');
/** Database Charset to use in creating database tables. */
/** The Database Collate type. Don't change this if in doubt. */
 * WordPress Database Table prefix.
 * You can have multiple installations in one database if you give each
root@a2dc2547ffe5:/var/www/html# exit
[root@ip-172-31-14-145 ec2-user]# docker exec -it database bash
root@083fd7d125de:/# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
root@083fd7d125de:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.46 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db                 |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

mysql> 


----



