# Práctia IOT - Home Assistant

## Objetivo:

Se instalará el software Home Assistant en un raspberry además de configurarlo para poder controlar un dispositivo desde este.
También se configurará el modulo de Google-Assistant para controlarlo por voz.


## Requisitos:

- Rasberry
- Protoboard
- Led
- Resistencia
- Google Home Mini

También es necesario contar con una cuenta de Google Cloud activa.

## Desarrollo:

### Instalación de Home Assistant

Para asegurar que todo funciona correctamente debemos tener insstalado la versión de Python 3.5.3
Nos conectamos a la Raspberry Pi por SSH. La contraseña predeterminada es `raspberry`.
Se necesitará habilitar acceso por SSH. El sitio de Raspberry Pi tiene las intrucciones [aquí](https://www.raspberrypi.org/documentation/remote-access/ssh/).

```bash
$ ssh pi@ipaddress
```

Con lo siguiente se puede cambiar la constraseña por defecto.

```bash
$ passwd
```

(Opcional) Actualizar el sistema.

```bash
$ sudo apt-get update
$ sudo apt-get upgrade -y
```

Instalar los siguientes paquetes.

```bash
$ sudo apt-get install python3 python3-venv python3-pip
```

Creamos una cuenta para Home Assistant llamada `homeassistant`.
Como este cuenta solamente la utilizaremos para correr Home Assistant el argumento extra de `-rm` es agregado para crear una cuenta de sistema y creamos un directorio de home.
Los argumentos de `-G dialout,gpio` agrega al usuario a los grupos de `dialout` y `gpio`. El primer grupo es requerido para usar controladores Z-Wave y Zigbee,
mientras que el segundo es requerido para comunicarse con el GPIO del Raspberry.

```bash
$ sudo useradd -rm homeassistant -G dialout,gpio
```

Next we will create a directory for the installation of Home Assistant and change the owner to the `homeassistant` account.

```bash
$ cd /srv
$ sudo mkdir homeassistant
$ sudo chown homeassistant:homeassistant homeassistant
```

Next up is to create and change to a virtual environment for Home Assistant. This will be done as the `homeassistant` account.

```bash
$ sudo -u homeassistant -H -s
$ cd /srv/homeassistant
$ python3 -m venv .
$ source bin/activate
```
Once you have activated the virtual environment (notice the prompt change) you will need to run the following command to install a required python package.

```bash
(homeassistant) homeassistant@raspberrypi:/srv/homeassistant $ python3 -m pip install wheel
```

Once you have installed the required python package it is now time to install Home Assistant!

```bash
(homeassistant) homeassistant@raspberrypi:/srv/homeassistant $ pip3 install homeassistant
```

Start Home Assistant for the first time. This will complete the installation for you, automatically creating the `.homeassistant` configuration directory in the `/home/homeassistant` directory, and installing any basic dependencies.

```bash
(homeassistant) $ hass
```
Tu puedes ahora alcanzar tu instalación en tu Raspberry Pi para la interfaz web en [http://ipaddress:8123](http://ipaddress:8123).

Cuando se ejecuta el comando `hass` por primera vez, este descargará , instalará y hará cache de las librecias/dependencias necesarias. Este procedimiento tomará entre 5 a 10 minutos. Durante ese tiempo, tu puedes obtener un error de "site cannot be reached" cuando accedes a la interfaz web.  Esto solamente ocurrirá la primera vez, y en los subsecuentes reinicios será mucho más rapido.

### Instalación de Remot3.it

Con este servicio obtendremos temporalmente una dirección URL, para acceder a nuestro Raspberry. Accedemos y creamos una cuenta en Remot3.it [aquí](https://www.remot3.it/web/index.html)

Decargamos el paquete de remot3.it weavedconnectd:

```bash
$ sudo apt-get install weavedconnectd
```

Configuramos weavedinstaller para configurar los servicios que añadiremos remot3.it. Ejecutamos

```bash
$ sudo weavedinstaller
```

Para lanzar el instalador interactivo.

