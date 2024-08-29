# Propuesta_IMS_KAMAILIO
Propuesta de conexion IMS to asterisk

### Para conectar tu P-CSCF (Proxy-Call Session Control Function) de Kamailio IMS a tu Asterisk, necesitas asegurarte de que ambos sistemas estén configurados correctamente para comunicarse entre sí. A continuación te recuerdo los pasos que debes seguir:

1. Configuración de Asterisk para conectarse al P-CSCF de Kamailio

Primero, necesitas configurar Asterisk para que se registre y pueda comunicarse con el P-CSCF.

Modificar sip.conf  en Asterisk
Dependiendo de si estás utilizando el módulo SIP (chan_sip)

Ejemplo para sip.conf (chan_sip):
```ini
[general]
context=default
allowoverlap=no
udpbindaddr=0.0.0.0:5060
tcpenable=no
transport=udp
srvlookup=yes
externip=YOUR_PUBLIC_IP
localnet=YOUR_LOCAL_NETWORK/255.255.255.0

register => <username>:<password>@172.22.0.21:5060/<extension>

[pcscf]
type=peer
host=172.22.0.21  # IP del P-CSCF de Kamailio
port=5060
context=from-pcscf
disallow=all
allow=ulaw
allow=alaw
insecure=invite,port
nat=yes
externip: Configura esto con la IP pública de tu servidor Asterisk si es necesario.
localnet: Configura la red local a la que pertenece tu servidor Asterisk.
register: Configura Asterisk para que se registre en el P-CSCF de Kamailio.
```

2. Configurar el P-CSCF (Kamailio) para reconocer Asterisk

Necesitas asegurarte de que Kamailio (actuando como P-CSCF) está configurado para aceptar y enrutar llamadas hacia y desde Asterisk.

Modificar kamailio_pcscf.cfg

Configurar las rutas SIP:

Asegúrate de que el archivo de configuración está configurado para enrutar las solicitudes SIP de Asterisk.
Añade una entrada en la sección de rutas para gestionar las solicitudes SIP hacia Asterisk.
```cfg
Copy code
route {
    # Forward INVITE requests to Asterisk
    if (is_method("INVITE") && $rd == "YOUR_DOMAIN") {
        rewritehostport("ASTERISK_IP:5060");
        t_relay();
        exit;
    }

    # Handle REGISTER requests for Asterisk
    if (is_method("REGISTER") && $rd == "YOUR_DOMAIN") {
        rewritehostport("ASTERISK_IP:5060");
        t_relay();
        exit;
    }

    # Other routing logic...
}
YOUR_DOMAIN: Reemplaza esto con el dominio que estás utilizando.
ASTERISK_IP: La IP de tu servidor Asterisk.
```        
Actualizar la lista de dispatchers:

Si estás utilizando el módulo de Dispatcher, asegúrate de añadir Asterisk en la lista.

```bash
echo "1 sip:ASTERISK_IP:5060 0" >> /etc/kamailio/dispatcher.list
```
3. Probar la Conexión

Reinicia Kamailio para aplicar los cambios:

```bash 
asterisk -rx "core restart now"
```

Verifica el registro:

Asegúrate de que Asterisk se registra correctamente en el P-CSCF.
Puedes usar el comando sip show peers (chan_sip) para verificar el estado.

Realiza una llamada de prueba:

Intenta realizar una llamada a través de Asterisk y verifica que la señalización SIP y el tráfico RTP fluyan correctamente.


    Configura Asterisk para que se registre y se comunique con el P-CSCF en Kamailio.

    Configura Kamailio para enrutar correctamente las llamadas hacia Asterisk.

    Reinicia ambos servicios y realiza pruebas para asegurarte de que la conexión funciona correctamente.

    Con estos pasos, deberías poder conectar tu P-CSCF de Kamailio IMS a tu servidor Asterisk sin problemas.
