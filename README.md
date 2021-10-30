# Jarkom-Modul-2-TI6-2021

## Anggota Kelompok T6:

- Syakhisk Al-Azmi 05311940000003
- Muhammad Rizqi Wijaya 05311940000014
- Shafira Firdausi 05311940000040

# Soal

### 1. EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha.

![Picture1.png](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Picture1.png)

Dibuat topologi seperti Gambar 1. Berikut IP yang dibutuhkan:

```bash
PREFIX="192.214"
DNS="192.168.122.1"
FOOSHA_e1_IP="$PREFIX.1.1"
FOOSHA_e2_IP="$PREFIX.2.1"
LOGUETOWN_IP="$PREFIX.1.2"
ALABASTA_IP="$PREFIX.1.3"
ENIESLOBBY_IP="$PREFIX.2.2"
ENIESLOBBY_IP_REV="$(echo $ENIESLOBBY_IP | sed -r 's/([^\.]*)\.([^\.]*)\.([^\.]*)\.([^\.]*)/\3\.\2\.\1/')"
PTR_RECORD="$ENIESLOBBY_IP_REV.in-addr.arpa"
WATER7_IP="$PREFIX.2.3"
SKYPIE_IP="$PREFIX.2.4"
```

### 2. Membuat website utama dengan mengakses **franky.ti6.com** dengan alias **www.franky.ti6.com** pada folder *****kaizoku**.***

Tambahkan zone **franky.ti6.com** pada `/etc/bind/named.conf.local` pada EniesLobby

```bash
zone "franky.ti6.com" {
	type master;
	file "/etc/bind/kaizoku/franky.ti6.com";
};
```

Lalu tambahkan folder *kaizoku,* dan edit konfigurasi **franky.ti6.com** di `/etc/bind/kaizoku/franky.ti6.com` dengan menambahkan **www** sebagai CNAME dari **franky.ti6.com** pada EniesLobby

```bash
\$TTL    604800
@       IN      SOA     franky.ti6.com. root.franky.ti6.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.ti6.com.
@       IN      A       $ENIESLOBBY_IP
www     IN      CNAME   franky.ti6.com.
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled.png)

### 3. Buat subdomain **super.franky.ti6.com** dengan alias **www.super.franky.ti6.com** yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

Tambahkan zone **super.franky.ti6.com** pada `/etc/bind/named.conf.local` pada EniesLobby.

```bash
zone "super.franky.ti6.com" {
	type master;
  	also-notify { ' $WATER7_IP '; };
  	allow-transfer { ' $WATER7_IP '; };
	file "/etc/bind/kaizoku/super.franky.ti6.com";
};
```

Tambahkan DNS record pada `/etc/bind/kaizoku/franky.ti6.com` pada EniesLobby.

```bash
super   IN      A       $SKYPIE_IP
```

Lalu edit konfigurasi **super.franky.ti6.com** di `/etc/bind/kaizoku/super.franky.ti6.com` pada EniesLobby.

```bash
\$TTL    604800
@       IN      SOA     super.franky.ti6.com. root.super.franky.ti6.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.ti6.com.
@       IN      A       $SKYPIE_IP
www     IN      CNAME   super.franky.ti6.com.
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%201.png)

### 4. Buat reverse domain untuk domain utama.

Tambahkan zone reverse pada `/etc/bind/named.conf.local` pada EniesLobby.

```bash
zone "'$PTR_RECORD'" {
	type master;
	file "/etc/bind/kaizoku/'$PTR_RECORD'";
};
```

Copykan `/etc/bind/db.local` ke `/etc/bind/kaizoku/$PTR_RECORD` pada EniesLobby lalu edit konfigurasinya. 

```bash
\$TTL    604800
@       IN      SOA     franky.ti6.com. root.franky.ti6.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
$PTR_RECORD.       IN      NS      franky.ti6.com.
2                  IN      PTR     franky.ti6.com. ; byte ke 4 dari $ENIESLOBBY_P
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%202.png)

### 5. Buat Water7 sebagai DNS Slave untuk domain utama.

Pada EniesLobby, tambahkan konfigurasi di `/etc/bind/named.conf.local` pada zone **franky.ti6.com.**

```bash
zone "franky.ti6.com" {
	type master;
  	also-notify { ' $WATER7_IP '; };
  	allow-transfer { ' $WATER7_IP '; };
	file "/etc/bind/kaizoku/franky.ti6.com";
};
```

Pada Water7, edit konfigurasi dengan menambahkan zone **franky.ti6.com** di `/etc/bind/named.conf.local` 

```bash
zone "franky.ti6.com" {
 type slave;
 masters { ' $ENIESLOBBY_IP '; };
 file "/var/lib/bind/franky.ti6.com";
};
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%203.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%204.png)

### 6. Setelah itu terdapat subdomain **mecha.franky.ti6.com** dengan alias **www.mecha.franky.ti6.com** yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder *sunnygo*.

Pada EniesLobby, tambahkan DNS record pada `/etc/bind/kaizoku/franky.ti6.com`

```bash
ns1     IN      A       $WATER7_IP     
mecha   IN      NS      ns1
```

Tambahkan `allow-query{any;};` di `/etc/bind/named.conf.options`

```bash
options {
        directory \"/var/cache/bind\";
        // forwarders {
        //      $DNS; //ns foosha
        // };
        // dnssec-validation auto;
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

Tambahkan juga 

Pada Water7, tambahkan zone **mecha.franky.ti6.com** di `/etc/bind/named.conf.local`

```bash
zone "mecha.franky.ti6.com" {
	type master;
	file "/etc/bind/sunnygo/mecha.franky.ti6.com";
};
```

Buat folder *sunnygo,* lalu edit konfigurasi di `/etc/bind/sunnygo/mecha.franky.ti6.com`

```bash
\$TTL    604800
@       IN      SOA     mecha.franky.ti6.com. root.mecha.franky.ti6.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      mecha.franky.ti6.com.
@       IN      A       $SKYPIE_IP
www     IN      CNAME   mecha.franky.ti6.com.
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%205.png)

### 7. Dibuatkan subdomain melalui Water7 dengan nama **general.mecha.franky.ti6.com** dengan alias **www.general.mecha.franky.ti6.com** yang mengarah ke Skypie.

Pada Water7, tambahkan zone pada `/etc/bind/named.conf.local` 

```bash
zone "general.mecha.franky.ti6.com" {
	type master;
	file "/etc/bind/sunnygo/general.mecha.franky.ti6.com";
};
```

Tambahkan DNS record pada `/etc/bind/sunnygo/mecha.franky.ti6.com` 

```bash
general IN      A       $SKYPIE_IP
```

Edit konfigurasi di `/etc/bind/sunnygo/general.mecha.franky.ti6.com` 

```bash
\$TTL    604800
@       IN      SOA     general.mecha.franky.ti6.com. root.general.mecha.franky.ti6.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      general.mecha.franky.ti6.com.
@       IN      A       $SKYPIE_IP
www     IN      CNAME   general.mecha.franky.ti6.com.
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%206.png)

### 8. Lakukan konfigurasi Webserver **www.franky.ti6.com** dengan DocumentRoot pada /var/www/franky.ti6.com

Pada Skypie, unduh file franky.zip dan pindah ke `/var/www/franky.ti6.com`

```bash
curl -Lk https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/raw/main/franky.zip -o franky.zip
unzip franky.zip
mv franky /var/www/franky.ti6.com
```

Buat VirtualHost dan tambahkan konfigurasi untuk franky.ti6.com pada `/etc/apache2/sites-available/franky.ti6.com.conf`.

```bash
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName franky.ti6.com
        ServerAlias www.franky.ti6.com
        DocumentRoot /var/www/franky.ti6.com

				<Directory /var/www/franky.ti6.com>
             AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%207.png)

### 9. URL **www.franky.ti6.com/index.php/home** dapat menjadi menjadi [](http://www.franky.com/home)**www.franky.ti6.com/home**

Pada Skypie, tambahkan syntax RewriteCond dan RewriteRule di `var/www/franky.ti6.com/.htaccess`

```bash
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*) /index.php/$1 [L]
</IfModule>
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%208.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%209.png)

### 10. Pada subdomain **www.super.franky.ti6.com** dibutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.ti6.com.

Pada Skypie, unduh file super.franky.zip dan pindahkan ke /var/www/super.franky.ti6.com

```bash
curl -Lk https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/raw/main/super.franky.zip -o super.franky.zip
unzip super.franky.zip
mv super.franky /var/www/super.franky.ti6.com
```

Buat VirtualHost dan tambahkan konfigurasi FollowSymLinks dan Multiviews untuk super.franky pada `/etc/apache2/sites-available/super.franky.ti6.com.conf`.

```bash
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName super.franky.ti6.com
        ServerAlias www.super.franky.ti6.com
        DocumentRoot /var/www/super.franky.ti6.com

         <Directory /var/www/super.franky.ti6.com>
             Options +FollowSymLinks -Multiviews
             AllowOverride All
         </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2010.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2011.png)

### 11. Tetapi pada folder /public, hanya dapat melakukan directory listing saja.

Pada Skypie, tambahkan konfigurasi Indexes masing-masing folder pada virtual host super.franky di  `/etc/apache2/sites-available/super.franky.ti6.com.conf`.

```bash
	<Directory /var/www/super.franky.ti6.com/public>
             Options +Indexes
             AllowOverride All
         </Directory>
         <Directory /var/www/super.franky.ti6.com/error>
             Options -Indexes
             AllowOverride All
         </Directory>
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2012.png)

### 12. Disiapkan error file 404.html pada folder /errors untuk mengganti error kode pada apache.

Pada Skypie, tambahkan konfigurasi ErrorDocument pada virtual host super.franky di  `/etc/apache2/sites-available/super.franky.ti6.com.conf`.

```bash
ErrorDocument 404 /error/404.html
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2013.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2014.png)

### 13. Dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset **www.super.franky.ti6.com/public/js** menjadi [](http://www.super.franky.com/js)**www.super.franky.ti6.com/js**

Pada Skypie, tambahkan konfigurasi Alias pada virtual host super.franky di  `/etc/apache2/sites-available/000-default.conf`.

```bash
Alias "/js" "/var/www/super.franky.ti6.com/public/js"
```

***Hasil Output:***

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2015.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2016.png)

### 14. Web [](http://www.mecha.franky.com/)**www.general.mecha.franky.ti6.com** hanya bisa diakses dengan port 15000 dan port 15500.

Pada Skypie, unduh file general.mecha.franky.zip dan pindahkan ke `/var/www/general.mecha.franky.ti6.com`

```bash
curl -Lk https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/raw/main/general.mecha.franky.zip -o general.mecha.franky.zip
unzip general.mecha.franky.zip
mv general.mecha.franky /var/www/general.mecha.franky.ti6.com
```

Tambahkan Listen kedua port di `/etc/apache2/ports.conf`

```bash
Listen 15000
Listen 15500
```

Buat VirtualHost dengan port 15000 dan 15500 untuk general.mecha.franky pada `/etc/apache2/sites-available/general.mecha.franky.ti6.com.conf`.

```bash
<VirtualHost *:15000 *:15500>
        ServerAdmin webmaster@localhost
        ServerName general.franky.ti6.com
        ServerAlias www.general.mecha.franky.ti6.com
        DocumentRoot /var/www/general.mecha.franky.ti6.com

				 <Directory /var/www/general.mecha.franky.ti6.com>
             AllowOverride All
         </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

***Hasil Output:***

- Berikut hasil jika mengakses melalui port general

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2017.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2018.png)

- Berikut hasil jika mengakses melalui port 15000 dan 15500

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2019.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2020.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2021.png)

### 15. **www.general.mecha.franky.ti6.com** hanya bisa diakses dengan authentikasi username *luffy* dan password *onepiece* dan file di /var/www/general.mecha.franky.ti6.

Pada Skypie, buat user luffy di `/var/www/general.mecha.franky.ti6` 

```bash
htpasswd -bc /var/www/general.mecha.franky.ti6 luffy onepiece
```

Lalu tambahkan location pada VirtualHost general.mecha.franky di `/etc/apache2/sites-available/000-default.conf`

```bash
	<Location />
		Deny from all
		AuthUserFile /var/www/general.mecha.franky.ti6
		AuthName "Restricted Area"
		AuthType Basic
		Satisfy Any
		require valid-user
	</Location>
```

***Hasil Output:***

- Mencoba akses **[www.general.mecha.franky.ti6.com](http://www.general.mecha.franky.ti6.com):15000 tanpa auth**

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2022.png)

- Masukkan username dan password

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2023.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2024.png)

- Berhasil mengakses **[www.general.mecha.franky.ti6.com](http://www.general.mecha.franky.ti6.com):15000**

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2023.png)

### 16. Setiap kali mengakses IP Skypie, akan diahlikan secara otomatis ke [](http://www.franky.com/)**[www.franky.ti6.com](http://www.franky.xxx.com/)**.

Tambahkan konfigurasi RewriteCond dan RewriteRule pada `/var/www/franky.ti6.com/.htaccess`

```bash
RewriteCond %{HTTP_HOST} ^192\.214\.2.4$
RewriteRule ^(.*)$ http://franky.ti6.com/$1 [L,R=301]
```

***Hasil Output:***

- Pertama coba lynx IP Skypie

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2025.png)

- Mencoba akses dan teredirect ke www.franky.ti6.com

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2026.png)

- Berikut tampilan dan informasi dari www.franky.ti6.com

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2027.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2028.png)

### 17. Pada website **www.super.franky.ti6.com**, diminta untuk mengganti request gambar yang memiliki substring “franky” untuk diarahkan menuju franky.png.

Tambahkan konfigurasi RewriteCond dan RewriteRule pada `/var/www/super.franky.ti6.com/.htaccess`

```bash
RewriteCond %{REQUEST_URI} .*franky.*\.(jpg|png)
RewriteCond %{REQUEST_URI} !/public/images/franky.png
RewriteRule ^(.*)/ /public/images/franky.png [L,R=301]
```

***Hasil Output:***

- Jika mencoba mengunduh car.jpg maka akan terunduh car.jpg

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2029.png)

- Jika mencoba mengunduh eyeoffranky.jpg maka akan terunduh franky.jpg

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2030.png)

![Untitled](Laporan%20Jarkom%2019777f3848be4b1da71407bef4406200/Untitled%2031.png)

# Kendala yang Dialami:

- Dua anggota kelompok tidak bisa melakukan apt update pada gns3nya.
- Beberapa soal yang sedikit ambigu sehingga dilakukan beberapa kali revisi setelah bertanya kepada asisten.
- Agak kesulitan di logic .htaccess pada nomor 17, namun akhirnya berhasil.
