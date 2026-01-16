# Máquina Upload (DockerLabs)

<img width="498" height="325" alt="Screenshot" src="https://github.com/user-attachments/assets/a4bc2f40-30a3-4602-9a37-0bc5a41b3972" />

## Fase de Reconocimiento
Con lo que empiezo siempre es con un escaneo de puertos rápido con NMAP:

```bash
> nmap -p- -n -v -Pn 172.17.0.2 -oN scan-basic
---
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
```bash
> nmap -p80 -sCV 172.17.0.2 -oN scan-full
---
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

- Puerto 80: Apache httpd 2.4.52

Con esta información me doy cuenta que lla versión del **Página WEB de Apache** es reciente. Así que procedo a inspeccionar un poco la página:

<img width="538" height="284" alt="Screenshot_1" src="https://github.com/user-attachments/assets/132d2d7e-5527-4fb3-bdfa-2f90c05c702f" />

Lo que se ve nada mas acceder a la página es un boton para subir archivos, con esto ya podemos pensar por donde va la cosa. Antes de intentar subir alguna cosa al servidor, decidor hacer un poco de `fuzzing` con `gobuster` con la intención de encontrar un **Directory Listing** donde se puedan ver los archivos subidos o algo por el estilo:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x html,php,txt,log > directory.txt
```

<img width="663" height="425" alt="Screenshot_3" src="https://github.com/user-attachments/assets/c61e2f50-3de2-4455-8278-210482a8ab59" />

Encontramos el payload llamado `/uploads` donde tiene toda la pinta que es lo que buscaba, un logar donde se puede ver los archivos subidos al servidor. Lo comprobamos y confirmamos que es así:

<img width="896" height="277" alt="Screenshot_4" src="https://github.com/user-attachments/assets/4b47ab19-1670-42df-b32e-735505e9bbd2" />
