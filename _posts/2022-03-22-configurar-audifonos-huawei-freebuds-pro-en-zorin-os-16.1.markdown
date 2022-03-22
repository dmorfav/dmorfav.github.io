### Configurar audífonos Huawei FreeBuds Pro en Zorin OS 16.1

Luego de haber adquirido unos audífonos Huawei FreeBuds Pro he decidido sacarle el máximo provecho emparejandolos con mi ordenador y poder utilizarlos como manos libre _(hands free)_ 
pero me he percartado que no me funcionaba el micro de los audífonos. Me he puesto a investigar y he encontrado la solución así que la voy a registrar para futuras instalaciones del **SO**.

### Obteniendo info del micro
Accedemos a una terminal y en ella ejecutamos ```bluetoothctl```

y obtendremos algo como esto

![image](https://raw.githubusercontent.com/dmorfav/dmorfav.github.io/main/images/posts/configurar-audifonos-huawei-freebuds-pro-en-zorin-os-16/159496234-9509bbdd-09be-44e5-b521-974df4e16f9c.png)

luego ejecutamos el comando ```info``` dentro de este apartado para verificar toda la info de los audífonos

### instalando el Pipewire

[pipewire-debian](https://pipewire-debian.github.io/pipewire-debian/)

```sh
sudo add-apt-repository ppa:pipewire-debian/pipewire-upstream && \
sudo apt update && \
sudo apt install pipewire && \
sudo apt install libspa-0.2-bluetooth && \
sudo apt install pipewire-audio-client-libraries
```
### Actualizando los servicios

Una vez actualizado el paquete desde el PPA vamos a actualizar los servicios de ```bluetooth``` y ```mask pulseaudio```

```sh
systemctl --user daemon-reload && \
systemctl --user --now disable pulseaudio.service pulseaudio.socket && \
systemctl --user mask pulseaudio && \
systemctl --user --now enable pipewire-media-session.service
```

Una vez realizado los cambios procedemos a reiniciar el ordenador para garantizar que todos los servicios se levanten con los cambios realizados

```sh
systemctl --user restart pipewire && sudo reboot
```
### Apreciando el resutado final
Una vez reiniciado el ordenador podemos apreciar que ya esta disponible el micro para ser utilizado

![image](https://raw.githubusercontent.com/dmorfav/dmorfav.github.io/main/images/posts/configurar-audifonos-huawei-freebuds-pro-en-zorin-os-16/159499403-5d324481-261c-48fa-a3d4-8ac1f10020db.png)

Lo único que no me ha hecho mucha gracia es que los audifonos pasan a utilizar el canal ```mono``` y no tengo idea de porque sucede.
