# Máquina Psycho (DockerLabs)
<img width="509" height="332" alt="Screenshot_9" src="https://github.com/user-attachments/assets/3b0f1593-0919-49f0-b1db-4e6da0881971" />

## Fase de Reconocimiento
Con lo que empiezo siempre es con un escaneo de puertos rápido con NMAP:

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
  
Me dispongo, entonces, a buscar información dentro de la **Página WEB de Apache** que contiene el servidor. No logro observar nada relevante, excepto el mensaje de error que hay al final de la página:

<img width="1197" height="540" alt="web" src="https://github.com/user-attachments/assets/69b0f766-78ad-4ed0-9ef6-3277298d417d" />

<img width="806" height="38" alt="error" src="https://github.com/user-attachments/assets/e77c68aa-c180-4a40-b2fd-7c83b7147426" />

