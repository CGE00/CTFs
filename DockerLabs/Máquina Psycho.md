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
nmap -p22,80 -sVC 172.17.0.2 -oN scan-full
---
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 38:bb:36:a4:18:60:ee:a8:d1:0a:61:97:6c:83:06:05 (ECDSA)
|_  256 a3:4e:4f:6f:76:f2:ba:50:c6:1a:54:40:95:9c:20:41 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: 4You
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kerne
```

- Puerto 22: OpenSSH 9.6p1 Ubuntu 3ubuntu13.4
- Puerto 80: Apache httpd 2.4.58

Con esta información me doy cuenta que las versiones de los servicios son relativamente recientes.  
  
Me dispongo, entonces, a buscar información dentro de la **Página WEB de Apache** que contiene el servidor. No logro observar nada relevante, excepto el mensaje de error que hay al final de la página:

<img width="1197" height="540" alt="web" src="https://github.com/user-attachments/assets/69b0f766-78ad-4ed0-9ef6-3277298d417d" />

<img width="806" height="38" alt="error" src="https://github.com/user-attachments/assets/e77c68aa-c180-4a40-b2fd-7c83b7147426" />

Después de buscar y probar ataques, decido hacer fuzzing para encontrar un parámetro oculto en la **URL**:
```bash
wfuzz -c --hh=2596 -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt -u "http://172.17.0.2/index.php?FUZZ=1”
---
000005057:   200        62 L     166 W      2582 Ch     "secret" 
```
- Parámetro oculto: "secret"

## Fase de Explotación
Una vez con toda la información recabada y con el parámetro, empiezo a probar comandas. Después varias pruebas, decubro que si ponemos la ruta de un fichero lo muestra por la página web. Decido mostrar directamente el archivo `passwd`:

<img width="958" height="875" alt="Screenshot_7" src="https://github.com/user-attachments/assets/41d9c052-9c5c-4c04-813d-54c6a3a86a9d" />
  
Encuentro dos usuarios que se podrían vulnerar **luisillo** y **vaxei**, así que decido utilizar `hydra` para hacer fuerza bruta al **servidor SSH** que hemos descubierto anteriormente pero no veo viable esta opción ya que no consigo sacar ninguna password.  
Decido investigar la clave privada de los usuarios del **servidor SSH**, buscando directamente el archivo `id_rsa` mediante el parámetro oculto en la **URL**:
  
<img width="953" height="875" alt="Screenshot_1" src="https://github.com/user-attachments/assets/4e10845c-44ab-4123-8ccf-a5b99be3569b" />
    
Decido copiarme desde el código fuente de la página web la clave privada y le doy permisos:

<img width="373" height="107" alt="Screenshot_2" src="https://github.com/user-attachments/assets/d22b3f12-eb3d-4f23-a378-f14d6a8e93a3" />

Con la clave privada podemos iniciar sesión en el **servidor SSH** saber el password del usuario **vaxei**:

<img width="625" height="240" alt="Screenshot_3" src="https://github.com/user-attachments/assets/503d6f87-30ce-4a06-907c-f05498a5eaef" />

Lo primero que hago cuando accedo a una máquina es comprobar que permisos `sudo` tiene el usuario y con este usuario descubro que puede ejecutar código `perl` pero como usuario **luisillo**, así que decido pivotar al usuario **luisillo** ya que no encuentro nada interesante con el usuario **vaxei**:

<img width="838" height="138" alt="Screenshot_4" src="https://github.com/user-attachments/assets/b4fa6c52-67ef-4c14-8762-7b564e808cec" />  
  
```bash
> sudo -u luisillo perl -e 'exec "/bin/sh";'
> whoami
---
luisillo
```
  
Como **luisillo** compruebo también los permisos `sudo` tiene el usuario y con este usuario descubro que puede ejecutar un script llamado `paw.py`:  

<img width="1019" height="117" alt="Screenshot_10" src="https://github.com/user-attachments/assets/7a4a78d9-de06-4869-8a03-fc932ff57519" />
  
Investigo que permisos tengo dentro de la carpeta `/opt/` y observo que tengo permisos de ecritura:  
  
<img width="418" height="103" alt="Screenshot_11" src="https://github.com/user-attachments/assets/62c0f20b-880d-4454-90e7-9e4f214f3158" />
  
Así que decido borrar ese archivo y crear otro llamado igual `/paw.py`, pero que al ejecutarse me convierta en `root`:  
  
<img width="1047" height="40" alt="Screenshot_12" src="https://github.com/user-attachments/assets/124718c8-1ab9-4316-a8f7-32ceadb70665" />
  
Procedo a ejecutar el nuevo archivo `/paw.py`:
```bash
> sudo python3 /opt/paw.py
> whoami
---
root
```
  
## Comentarios
Esta máquina ha sido hasta la fecha la que mas tiempo le he tenido que dedicar, he investigado bastante por mi cuenta formas de vulnerar la máquina y eso me ha venido bien para ver formas de vulnerar sistemas. Pienso que la parte del `fuzzing` me vendrá bien lo que he aprendido para próximos **CTFs**. También he tenido la ayuda de mi profesor de **Seguridad** y **Servicios** en el Grado Superior de **ASIR** [ViejoFraile](https://www.youtube.com/@ViejoFraile), me ha explicado bien todas las vulnerabilidades que he realizado y eso me ha echo entederlo todo a mayor profundidad.

