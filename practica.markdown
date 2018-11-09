# Práctica IOT - Home Assistant

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

ou need to reload `systemd` to make the daemon aware of the new configuration.

```bash
$ sudo systemctl --system daemon-reload
```

To have Home Assistant start automatically at boot, enable the service.

```bash
$ sudo systemctl enable home-assistant@homeassistant
```

To disable the automatic start, use this command.

```bash
$ sudo systemctl disable home-assistant@homeassistant
```

To start Home Assistant now, use this command.
```bash
$ sudo systemctl start home-assistant@homeassistant
```

You can also substitute the `start` above with `stop` to stop Home Assistant, `restart` to restart Home Assistant, and 'status' to see a brief status report as seen below.

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

To get Home Assistant's logging output, simple use `journalctl`.

```bash
$ sudo journalctl -f -u home-assistant@homeassistant
```

### Raspberry Pi GPIO Switch

The `rpi_gpio` switch platform allows you to control the GPIOs of your [Raspberry Pi](https://www.raspberrypi.org/).

To use your Raspberry Pi's GPIO in your installation, add the following to your `configuration.yaml` file:

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
    3. Crear una Action, bajo sección build. Agregar la  URL de Home Assistant : `https://[YOUR HOME ASSISTANT URL:PORT]/api/google_assistant`, rempla the `[YOUR HOME ASSISTANT URL:PORT]` with the domain / IP address and the port under which your Home Assistant is reachable.
    4. Click `Done`. Then click on `Overview`, which will lead you back to the app details screen.
2. `Account linking` is required for your app to interact with Home Assistant. Set this up under the `Quick Setup` section.
    1. Leave it at the default `No, I only want to allow account creation on my website` and select Next.
    2. For the `Linking type` select `OAuth` and `Authorization Code`.
    3. Client ID: `https://oauth-redirect.googleusercontent.com/`, the trailing slash is important.
    4. Client Secret: Anything you like, Home Assistant doesn't need this field.
    5. Authorization URL (replace with your actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/auth/authorize`.
    6. Token URL (replace with your actual URL): `https://[YOUR HOME ASSISTANT URL:PORT]/auth/token`.
    7. Configure your client. Add scopes for `email` and `name`.
    8. Do **NOT** check `Google to transmit clientID and secret via HTTP basic auth header`.
    9. Testing instructions: Enter anything. It doesn't matter since you won't submit this app.

    <img src='/images/components/google_assistant/accountlinking.png' alt='Screenshot: Account linking'>

3. Back on the overview page. Click `Simulator` under `TEST`. It will create a new draft version Test App. You don't have to actually test, but you need to generate this draft version Test App.
4. If you haven't already added the component configuration to `configuration.yaml` file and restarted Home Assistant, you'll be unable to continue until you have.
5. Open the Google Assistant app and go into `Settings > Home Control`.
6. Click the `+` sign, and near the bottom, you should have `[test] your app name`. Selecting that should lead you to a browser to login your Home Assistant instance, then redirect back to a screen where you can set rooms for your devices or nicknames for your devices.
<p class='note'>
If you've added Home Assistant to the home screen, you have to first remove it from home screen, otherwise, this HTML5 app will show up instead of a browser. Using it would prevent Home Assistant to redirect back to the `Google Assistant` app.
</p>
7. If you want to allow other household users to control the devices:
    1. Go to the settings for the project you created in the [Actions on Google console](https://console.actions.google.com/).
    2. Click `Test -> Simulator`, then click `Share` icon in the right top corner. Follow the on-screen instruction:
        1. Add team members: Got to `Settings -> Permission`, click `Add`, type the new user's e-mail address and choose `Project -> Viewer` role.
        2. Copy and share the link with the new user.
        3. New user clicks the link with their own Google account, it will enable our draft test app under their account.
    3. Have the new user go to their `Google Assistant` app to add `[test] your app name` to their account.
8. If you want to use the `google_assistant.request_sync` service, to update devices without unlinking and relinking, in Home Assistant, then enable Homegraph API for your project:
    1. Go to the [Google API Console](https://console.cloud.google.com/apis/api/homegraph.googleapis.com/overview).
    2. Select your project and click Enable Homegraph API.
    3. Go to Credentials, which you can find on the left navigation bar under the key icon, and select API Key from Create Credentials.
    4. Note down the generated API Key and use this in the configuration.

## {% linkable_title Configuration %}

Now add the following lines to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
google_assistant:
  project_id: YOUR_PROJECT_ID
  api_key: YOUR_API_KEY

