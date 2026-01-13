# Máquina Injection (DockerLabs)

<img width="491" height="315" alt="Screenshot_9" src="https://github.com/user-attachments/assets/2cc43073-cfa4-49f6-ac63-c4171490b8cd" />

## Fase de Reconocimiento
Con lo que empiezo siempre es con un escaneo de puertos rápido con NMAP:

```bash
nmap -p- -n -v -Pn 172.17.0.2 -oN scan-basic
---
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Puedo observar que tanto el puerto 22 (SSH) y el puerto 80 (http) están abiertos. Lo siguiente que me propongo es obtener mas información de esos puertos, los servicios y versiones:
```bash
nmap -p80,22 -sCV 172.17.0.2 -oN scan-full 
---
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Iniciar Sesi\xC3\xB3n
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

- Puerto 22: OpenSSH 8.9p1 Ubuntu 3ubuntu0.6
- Puerto 80: Apache httpd 2.4.52

Con esta información me doy cuenta que las versiones de los servicios son relativamente recientes.Me dispongo, entonces, a buscar información dentro de la **Página WEB de Apache** que contiene el servidor.  

<img width="531" height="377" alt="Screenshot_1" src="https://github.com/user-attachments/assets/599fa844-eb75-4e4e-9db5-484ad4b369af" />

Como la máquina se llama **Injection** y mi intención era realizar una máquina donde se explotara una **Injección SQL**, decido introducir código `sql` directamente en el login que nos dispone la máquina jugando con las comillas y nos devuelve el siguiente mensaje:

<img width="961" height="434" alt="Screenshot_2" src="https://github.com/user-attachments/assets/bb17538a-b049-4a20-8790-f6aab06aa1e7" />

## Fase de Explotación

Con el mensaje que nos muestra podemos confirmar que el login lee `sql` y nos podemos aprovechar. Decido introducir una de las comandas mas populares y sencillas en **Injecciones SQL**:

```bash
'or true -- -
```

Y al introducir ese código `sql`, no lleva a la siguiente página web con la siguiente información:

<img width="676" height="162" alt="Screenshot_3" src="https://github.com/user-attachments/assets/12828dd7-91df-4e9e-9290-28e0fd319814" />

