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

## Fase de Explotación
Una vez recopilado toda la anterior información de la página, decido porceder a intentar distintos archivos para comprobar si hay alguna restricción con algún tipo de archivo en concreto. Después de varias pruebas, intento subir un archivo `.php` para ver que sucede:  
*(El archivo)*

<img width="385" height="81" alt="Screenshot_2" src="https://github.com/user-attachments/assets/d09efca2-2ead-4925-80c2-83250355b541" />

Compruebo si aparece en el **Directory Listing** y si lo puedo ejecutar:

<img width="919" height="120" alt="Screenshot_5" src="https://github.com/user-attachments/assets/c2b6ec77-fa32-40dd-93aa-d9918584a949" />

Puedo ver que se sube y se ejecuta sin problemas. Así que procedo a buscar un código para obtener una `reverseshell` con `php`. La que mas me han recomendado es la de [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell) y es la que utilizo en esta máquina, pero antes le indico mi ip:

<img width="355" height="186" alt="Screenshot_6" src="https://github.com/user-attachments/assets/26706929-c66d-4a61-a4ab-ebfa2867d573" />

Me pongo a la escucha con `netcat`:

```bash
> nc -lvp 1234
---
listening on [any] 1234 ...
```

Subo la **Reverseshell** al servidor y la ejecuto desde el **Directory Listing**:

<img width="938" height="322" alt="Screenshot_7" src="https://github.com/user-attachments/assets/43f0c893-84e9-47e6-9e65-fba24c426174" />

<img width="1031" height="212" alt="Screenshot_8" src="https://github.com/user-attachments/assets/a90e4a0c-70ae-4665-94a0-694dce8be43c" />
