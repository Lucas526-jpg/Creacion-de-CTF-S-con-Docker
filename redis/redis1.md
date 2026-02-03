# 🚩 CTF : Redis "Muy facil"

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-yellow?style=for-the-badge&logo=Redis&logoColor=black)
![Security](https://img.shields.io/badge/Security-CTF-red?style=for-the-badge)

Este proyecto documenta la creación de un CTF de nivel muy facil, explotando una configuración insegura en el servicio **Redis** sobre un contenedor **Ubuntu**.

---

## 📋 Prerrequisitos

* **Docker** instalado en tu máquina principal
* **redis-cli** y **Nmap** (para la fase de prueba)

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

ahora intalamos el servicio redis:

```bash
apt install redis-server -y
```

como un paso extra, podemos intalar nano para modificar el archivo de configuracion de redis, aunque tambien se puede hacer con cat, pero para aprender recomiendo nano:

```bash
apt install nano -y
```

### 5. Configurar el Servicio Redis
editaremos nuestro archivo de configuracion, esta en la ruta "/etc/redis/redis.conf", con el comando:

```bash
nano /etc/redis/redis.conf
```

buscaremos en el arhivo, la linea:

```bash
bind 127.0.0.1
```

y la cambiaremos por:

```bash
bind 0.0.0.0
```

ademas modificaremos el protected mode que por defecto viene activado, lo desactivamos:

```bash
protected-mode no
```

guardamos y cerramos el archivo

### 6. Iniciar el Servicio y poner la Bandera

lo iniciamos con el siguiente comando:

```bash
service redis-server start
```

ahora, accedemos al servicio:

```bash
redis-cli
```

para añadir la bandera, usaremos el comando set, de la siguiente forma:

```bash
set flag "flag{prueba_lol}"
```

luego salimos con el comando exit:

```bash
exit
```

IMPORTANTE: Antes de salir, debemos guardar los cambios en el disco duro, ya que Redis trabaja en la memoria RAM. Si no ejecutamos este comando, la bandera se borrará al reiniciar el contenedor, con el siguiente comando:

```bash
redis-cli save
```

## 🕵️ Fase 2: Prueba
antes de cerrar todo, verificamos que la vulnerabilidad sea accesible desde afuera

averiguamos la IP del contenedor (dentro de la terminal del contenedor):

```bash
hostname -I
```

desde nuestra máquina principal (otra terminal), escaneamos esa IP para confirmar que el puerto está abierto:

```bash
# reemplaza la ip por la que obtuviste en el paso anterior
nmap -p 6379 172.17.0.2
```

## Fase 3: Terminar las Configuraciones
una vez verificado, apagamos el servicio para guardar los datos correctamente en el disco:

```bash
service redis-server stop
```

### Creacion de la imagen y exportarlo como .tar