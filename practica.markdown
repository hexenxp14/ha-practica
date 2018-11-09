# Práctica IOT - Home Assistant

## Objetivo:

Se instalará el software Home Assistant en un raspberry además de configurarlo para poder controlar un dispositivo desde este.
También se configurará el modulo de Google-Assistant para controlarlo por voz.


## Requisitos:

- Rasberry
- Protoboard
- Led
- Resistencia
- Google Home Mini ó bien Celular con aplicación Google Assistant

También es necesario contar con una cuenta de Google Cloud activa y Remot3.it.

## Desarrollo:

### Instalación de Home Assistant

Para asegurar que todo funciona correctamente debemos tener insstalado la versión de Python 3.5.3
Nos conectamos a la Raspberry Pi por SSH. La contraseña predeterminada es `raspberry`.
Se necesitará habilitar acceso por SSH. El sitio de Raspberry Pi tiene las intrucciones [aquí](https://www.raspberrypi.org/documentation/remote-access/ssh/).

```bash
$ ssh pi@ipaddress
```

Con lo siguiente se puede cambiar la contraseña por defecto.

```bash
$ passwd
```

(Deseable) Actualizar el sistema.

```bash
$ sudo apt-get update
$ sudo apt-get upgrade -y
```

Instalar los siguientes paquetes.

```bash
$ sudo apt-get install python3 python3-venv python3-pip
```

Creamos una cuenta para Home Assistant llamada `homeassistant`.
Dado que esta cuenta solamente la utilizaremos para correr Home Assistant, el argumento extra de `-rm` es agregado para crear una cuenta de sistema y crear su directorio en home.
Los argumentos de `-G dialout,gpio` agrega al usuario a los grupos de `dialout` y `gpio`. El primer grupo es requerido para usar controladores Z-Wave y Zigbee, mientras que el segundo es requerido para comunicarse con el GPIO del Raspberry.

```bash
$ sudo useradd -rm homeassistant -G dialout,gpio
```

A continuación creamos un directorio para la instalación de Home Assistant y le cambiamos el propietario a `homeassistant`.

```bash
$ cd /srv
$ sudo mkdir homeassistant
$ sudo chown homeassistant:homeassistant homeassistant
```

Luego creamos y nos cambiamos a el virtual environment para Home Assistant. Esto será hecho desde la cuenta de `homeassistant`.

```bash
$ sudo -u homeassistant -H -s
$ cd /srv/homeassistant
$ python3 -m venv .
$ source bin/activate
```
Una vez que hemos activado el virtual environment (notar el cambio del prompt) necesitarás correr el siguiente comando para instalar un paquete requerido de python.

```bash
(homeassistant) homeassistant@raspberrypi:/srv/homeassistant $ python3 -m pip install wheel
```

Posterior de haber instalado este paquete requerido proseguimos ahora a instalar Home Assistant:

```bash
(homeassistant) homeassistant@raspberrypi:/srv/homeassistant $ pip3 install homeassistant
```

Iniciamos Home Assistant por primera vez. Esto finalizará la instalación por ti, creando automaticamente el directorio de configuración `.homeassistant` en el directorio `/home/homeassistant`, y instalando cualquier dependencia básica.

```bash
(homeassistant) $ hass
```
Tu puedes ahora alcanzar tu instalación en tu Raspberry Pi por la interfaz web en [http://ipaddress:8123](http://ipaddress:8123).

Cuando se ejecuta el comando `hass` por primera vez, este descargará , instalará y hará cache de las librecias/dependencias necesarias. Este procedimiento tomará entre 5 a 10 minutos. Durante ese tiempo, tu puedes obtener un error de "site cannot be reached" cuando accedes a la interfaz web.  Esto solamente ocurrirá la primera vez, y en los subsecuentes reinicios será mucho más rapido.

### Instalación de Remot3.it

Con este servicio obtendremos temporalmente una dirección URL, para acceder a nuestro Raspberry. Creamos una cuenta en Remot3.it [aquí](https://www.remot3.it/web/index.html)

Decargamos el paquete de remot3.it weavedconnectd:

```bash
$ sudo apt-get install weavedconnectd
```

Configuramos weavedinstaller para configurar los servicios que añadiremos remot3.it. Ejecutamos

```bash
$ sudo weavedinstaller
```

Para lanzar el instalador interactivo.

### Autoinicio usando systemd

La más nuevas distribuciones de Linux están tendiendo a usar `systemd` para la gestión de demonios. Un archivo de servicio es necesario para controlar Home Assistant con `systemd`. La plantilla abajo debe ser creada usando un editor de texto. Notar, que permisos de root vía `sudo` serán necesarios.

Para crear el archivo `sudo nano -w [filename]` puede ser usado. `[filename]` se remplaza con la ruta completa hacia el archivo. Así en nuestro caso será:

```bash
$ sudo nano -w /etc/systemd/system/home-assistant@homeassistant.service`. 
```

Pegamos el siguiente texto:

```
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"

[Install]
WantedBy=multi-user.target
```

Presionamos CTRL-X luego Y para guardar y salir.

Debemos recarcar `systemd` para advertir al demonio de la nueva configuración.

```bash
$ sudo systemctl --system daemon-reload
```

Para que Home Assistant inicie automáticamente al encender, habilitamos el servicio.

```bash
$ sudo systemctl enable home-assistant@homeassistant
```

Para deshabilitar el inicio automático, usar este comando.

```bash
$ sudo systemctl disable home-assistant@homeassistant
```

Para iniciar  Home Assistant ahora, usar este comando.
```bash
$ sudo systemctl start home-assistant@homeassistant
```
Se puede también sustiuir el `start` de arriba con `stop` para detener Home Assistant, `restart` para reiniciar Home Assistant, y a 'status' para ver un reporte de estado.

```bash
$ sudo systemctl status home-assistant@homeassistant.service
● home-assistant@fab.service - Home Assistant for homeassistant
   Loaded: loaded (/etc/systemd/system/home-assistant@homeassistant.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2016-03-26 12:26:06 CET; 13min ago
 Main PID: 30422 (hass)
   CGroup: /system.slice/system-home\x2dassistant.slice/home-assistant@homeassistant.service
           ├─30422 /usr/bin/python3 /usr/bin/hass
           └─30426 /usr/bin/python3 /usr/bin/hass
[...]
```

Para obtener la salida de logging, simplemente usamos `journalctl`.

```bash
$ sudo journalctl -f -u home-assistant@homeassistant
```

### Raspberry Pi GPIO Switch

El componente `rpi_gpio` switch nos permite controlar las GPIOs del [Raspberry Pi](https://www.raspberrypi.org/).

Para usar este en nuestra instalación, agregamos lo siguiente al archivo `configuration.yaml`:

```yaml
# Example configuration.yaml entry
switch:
  - platform: rpi_gpio
    ports:
      18: Led
```

### Google Assistant

El componente `google_assistant` nos permite controla cosas via Google Assistant (en tu celular o tablet) o un dispositivo Google Home.

Tu necesitas crear un API Key con la [Google Cloud API Console](https://console.cloud.google.com/apis/api/homegraph.googleapis.com/overview) que nos permitirá actualizar dispositivos sin desanlazar y reelanzar la cuenta. Si no se provee una, el servicio `google_assistant.request_sync` no es expuesto. Es recomendable  configurar esto llave ya que nos permite el uso del comando, "Ok Google, sincroniza luces". Una vez establecido, tu puedes llamar a este servicio (o comando) cada ve que se agregue un nuevo dispositivo que se deseee controlar vía la intregración con Google Assistant.

1. Crear un nuevo proyecto en [Actions on Google console](https://console.actions.google.com/).
    1. Agregar/Importar un proyecto y darle un nombre.
    2. Clic en la tarjeta `Home Control`, seleccionar la recomendación `Smart home`.
    3. Crear una Action, bajo la sección build. Agregar la  URL de Home Assistant : `https://[YOUR HOME ASSISTANT URL:PORT]/api/google_assistant`, remplazar '[YOUR HOME ASSISTANT URL:PORT]` con el dominio ó dirección ip  y el puerto bajo en el que tu Home Assistant es alcanzable.
    4. Clic `Done`. Entonces clic en `Overview`, que te regresará a la pantalla de detalles de la app.
2. `Account linking` es requerido para que tu app interactue con Home Assistant. Configuramos esto bajo la seección de `Quick Setup`.
    1. Dejar el valor predeterminado `No, I only want to allow account creation on my website` y clic Next.
    2. Para `Linking type` seleccionar `OAuth` y `Authorization Code`.
    3. Client ID: `https://oauth-redirect.googleusercontent.com/`, el trailing slash es importante.
    4. Client Secret: Cualquier cosa, Home Assistant no necesita este campo.
    5. Authorization URL (reemplazar con tu actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/auth/authorize`.
    6. Token URL (remplazar con tu actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/auth/token`.
    7. Configurar tu cliente. Agregar scopes para `email` y `name`.
    8. **NO** marcar `Google to transmit clientID and secret via HTTP basic auth header`.
    9. Testing instructions: Introducir cualquier cosa. No importa dado que no aplicaremos esta app.

3. Regresamos a la página de overview. Clic en `Simulator` bajo `TEST`. Creará una nueva versión borrador Test App. No necesitamos probarla , pero si necesitamos generar esta versión de borrador de Test App.
4. Si tu no has agregado el compoenente al archivo de `configuration.yaml` y reiniciado Home Assistant, no se debe continuar hasta tenerlo.
5. Abrimos la plicación Google Assistant y vamos a `Settings > Home Control`.
6. Clic en el signo `+`, y en la parte de hasta arriba, debes tener `[test] tu aplicación de prueba`. Al Seleccionar esto debería irse a un navegador web en la pagina de inicio de tu instancia de Home Assistant, entonces te redirige a una pantalla donde tu puedes establece habitaciones para tus dispositivos y apodos.
7. Si tu quieres usar el servicio `google_assistant.request_sync`, para actualizar los dispositivos sin tener que desenlazar y reenlazar, en Home Assistant, entonces habilitamos el Homegraph API para tu proyecto:
    1. Ir a la [Google API Console](https://console.cloud.google.com/apis/api/homegraph.googleapis.com/overview).
    2. Seleccionar tu proyecto y clic Enable Homegraph API.
    3. Ir a Credentials, que tu puedes encontrar en la barra de navegación izquierda bajo el icono de llave, y seleccionamos API Key desde Create Credentials.
    4. Vemos se genera la API Key  y usamos esta en la configuración.

#### Configuración

Agregar las siguientes líneas a tu archivo `configuration.yaml`:

```yaml
# Example configuration.yaml entry
google_assistant:
  project_id: YOUR_PROJECT_ID
  api_key: YOUR_API_KEY
```
