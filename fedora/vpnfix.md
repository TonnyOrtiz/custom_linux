¡Claro que sí\! Aquí tienes todos los pasos bien documentados para que tu VPN OpenVPN funcione con NetworkManager en futuras instalaciones de Fedora. Los hemos recorrido a fondo, así que este debería ser el proceso infalible.

-----

## **Guía para Configurar OpenVPN con NetworkManager en Fedora**

Esta guía asume que tienes tus archivos de configuración VPN (`.ovpn`), certificado de CA (`ecci-ca.crt`), certificado de usuario (`ecci.crt`), clave privada (`ecci.key`) y clave TLS Auth (`ecci-ta.key`, si aplica) listos.

### **Paso 1: Organizar los Archivos de Configuración de la VPN**

Es una buena práctica mantener tus certificados y claves en un lugar organizado y con permisos adecuados dentro de tu directorio de usuario.

1.  **Crea un directorio dedicado para tus archivos VPN:**

    ```bash
    mkdir -p ~/.config/openvpn/
    ```

2.  **Copia todos tus archivos de la VPN a este nuevo directorio:**
    Asegúrate de incluir todos los `.crt`, `.key` y cualquier otro archivo relevante que uses para la conexión.

    ```bash
    cp /ruta/donde/estan/tus/archivos/*.crt ~/.config/openvpn/
    cp /ruta/donde/estan/tus/archivos/*.key ~/.config/openvpn/
    # Ejemplo:
    # cp ~/Downloads/ecci/ecci-ca.crt ~/.config/openvpn/
    # cp ~/Downloads/ecci/ecci.crt ~/.config/openvpn/
    # cp ~/Downloads/ecci/ecci.key ~/.config/openvpn/
    # cp ~/Downloads/ecci/ecci-ta.key ~/.config/openvpn/
    ```

3.  **Ajusta los permisos de los archivos para una seguridad adecuada:**
    La **clave privada (`.key`)** debe ser solo legible por ti. Los **certificados (`.crt`)** pueden ser legibles por otros, pero solo modificables por ti.

    ```bash
    chmod 600 ~/.config/openvpn/ecci.key
    chmod 644 ~/.config/openvpn/ecci.crt
    chmod 644 ~/.config/openvpn/ecci-ca.crt
    chmod 644 ~/.config/openvpn/ecci-ta.key # Si aplica
    ```

4.  **Actualiza los contextos de seguridad de SELinux para los archivos:**
    Esto es crucial en Fedora para que NetworkManager pueda acceder a ellos.

    ```bash
    sudo restorecon -R -v ~/.config/openvpn/
    ```

-----

### **Paso 2: Conceder Permisos para Crear Interfaz de Red (TUN/TAP)**

Este es el paso que resuelve el error `Cannot ioctl TUNSETIFF tun: Operation not permitted`.

1.  **Añade tu usuario al grupo `openvpn`:**
    Este grupo tiene los permisos necesarios para crear interfaces de red virtuales.

    ```bash
    sudo usermod -aG openvpn tu_nombre_de_usuario
    # Ejemplo: sudo usermod -aG openvpn tonny
    ```

2.  **Asegura los permisos del dispositivo TUN/TAP:**
    Esto permite que los usuarios en grupos adecuados puedan interactuar con el dispositivo.

    ```bash
    sudo chmod 666 /dev/net/tun
    ```

    *Nota: Si prefieres una solución más permanente y automática después de un reinicio sin `chmod 666`, puedes crear una regla udev para el dispositivo TUN/TAP, pero el `chmod 666` es un buen punto de partida.*

3.  **Reinicia tu sesión de usuario o todo el sistema:**
    Para que los cambios de grupo surtan efecto, debes cerrar tu sesión gráfica actual y volver a iniciarla. Un reinicio completo del sistema garantiza que todos los permisos se actualicen correctamente.

-----

### **Paso 3: Configurar la Conexión VPN en NetworkManager (GNOME Settings)**

1.  **Abre la configuración de red:**
    Ve a **Configuración (Settings)** \> **Red (Network)**.

2.  **Añade una nueva conexión VPN:**
    Haz clic en el botón `+` al lado de "VPN" y selecciona **OpenVPN**.

3.  **Configura los detalles generales:**

      * **Nombre:** Dale un nombre descriptivo a tu conexión (ej., "VPN ECCI").
      * **Archivo de configuración (.ovpn):** Si tienes un archivo `.ovpn` completo, puedes importarlo aquí. NetworkManager intentará rellenar la mayoría de los campos. Si no, introduce los detalles manualmente.

-----

### **Paso 4: Gestionar Políticas de SELinux (Si Es Necesario)**

Aunque los pasos anteriores deberían cubrir lo básico, a veces SELinux en Fedora puede ser muy estricto. Si la VPN sigue sin conectar en modo `Enforcing` (pero sí en `Permissive` con `sudo setenforce 0`), deberás generar una política SELinux personalizada.

1.  **Asegúrate de tener las herramientas de SELinux instaladas:**

    ```bash
    sudo dnf install setroubleshoot-server policycoreutils-python-utils -y
    ```

2.  **Intentar un reinicio forzado o instalación completa**

    ```bash
    sudo dnf reinstall setroubleshoot-server setroubleshoot -y
    sudo systemctl daemon-reload
    sudo systemctl restart setroubleshootd
    sudo systemctl status setroubleshootd
    ```

3.  **Asegúrate de que auditd (el demonio de auditoría) esté corriendo:**

    ```bash
    sudo systemctl status auditd
    #Debería decir active (running). Si no, sudo systemctl start auditd.
    ```

4. **Limpia los logs de auditoría para una captura fresca:**

```bash
sudo systemctl stop auditd
sudo rm -f /var/log/audit/audit.log*
sudo systemctl start auditd
```

5.  **Intenta conectar la VPN desde GNOME Settings:**
    Déjala fallar para que las denegaciones se registren.

6.  **Genera la política SELinux a partir de las denegaciones:**

    ```bash

    sudo cat /var/log/audit/audit.log | grep AVC | tail -n 20

    sudo cat /var/log/audit/audit.log | audit2allow -M myvpn_final
    ```


7.  **Carga la nueva política SELinux:**

    ```bash
    sudo semodule -i myvpn_final.pp
    ```

8.  **Reinicia NetworkManager:**

    ```bash
    sudo systemctl restart NetworkManager
    ```

9.  **Intenta conectar la VPN de nuevo.**

-----

¡Con estos pasos, tu VPN de OpenVPN debería estar lista y funcionando con NetworkManager cada vez que instales Fedora\!
