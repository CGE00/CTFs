# Máquina Upload (DockerLabs)

<img width="498" height="325" alt="Screenshot" src="https://github.com/user-attachments/assets/a4bc2f40-30a3-4602-9a37-0bc5a41b3972" />

Fase de Reconocimiento
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
