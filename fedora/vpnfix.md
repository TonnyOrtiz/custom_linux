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

4.  **Configura la identidad (certificados y claves):**

      * Ve a la pestaña **"Identidad" (Identity)**.
      * **Certificado de CA (CA Certificate):** Haz clic en el icono de la carpeta y navega hasta `~/.config/openvpn/ecci-ca.crt`.
      * **Certificado de usuario (User Certificate):** Haz clic en el icono de la carpeta y navega hasta `~/.config/openvpn/ecci.crt`.
      * **Clave privada (Private Key):** Haz clic en el icono de la carpeta y navega hasta `~/.config/openvpn/ecci.key`.
      * Ingresa tu **nombre de usuario y contraseña** para la autenticación de usuario si es requerida.

5.  **Configura las opciones TLS (si usas `tls-auth`):**

      * Haz clic en el botón **"Avanzado..." (Advanced...)**.
      * Ve a la pestaña **"Configuración TLS" (TLS Settings)**.
      * **Modo TLS (TLS Mode):** Selecciona **"Autenticación TLS" (TLS authentication)**.
      * **Clave de autenticación TLS (TLS authentication key):** Haz clic en el icono de la carpeta y navega hasta `~/.config/openvpn/ecci-ta.key` (si tienes este archivo).
      * **Dirección de clave (Key direction):** Si tu archivo `.ovpn` tenía `tls-auth ecci-ta.key 1`, selecciona **"1 (Response)"**. Si era `tls-auth ecci-ta.key 0`, selecciona "0 (Client)".

6.  **Guarda la configuración:**
    Haz clic en **"Aceptar" (OK)** en la ventana de configuración avanzada, y luego en **"Aplicar" (Apply)** o **"Guardar" (Save)** en la ventana principal de la VPN.

-----

### **Paso 4: Gestionar Políticas de SELinux (Si Es Necesario)**

Aunque los pasos anteriores deberían cubrir lo básico, a veces SELinux en Fedora puede ser muy estricto. Si la VPN sigue sin conectar en modo `Enforcing` (pero sí en `Permissive` con `sudo setenforce 0`), deberás generar una política SELinux personalizada.

1.  **Asegúrate de tener las herramientas de SELinux instaladas:**

    ```bash
    sudo dnf install setroubleshoot-server policycoreutils-python-utils -y
    ```

2.  **Limpia el log de auditoría para una captura fresca:**

    ```bash
    sudo truncate -s 0 /var/log/audit/audit.log
    ```

3.  **Asegúrate de que SELinux está en modo `enforcing`:**

    ```bash
    sudo setenforce 1
    ```

4.  **Intenta conectar la VPN desde GNOME Settings:**
    Déjala fallar para que las denegaciones se registren.

5.  **Genera la política SELinux a partir de las denegaciones:**

    ```bash
    sudo cat /var/log/audit/audit.log | grep "AVC denied" | audit2allow -M myvpn_policy_name
    ```

    Reemplaza `myvpn_policy_name` con un nombre único para tu política (ej., `ecci_vpn_policy`).

6.  **Carga la nueva política SELinux:**

    ```bash
    sudo semodule -i myvpn_policy_name.pp
    ```

7.  **Reinicia NetworkManager:**

    ```bash
    sudo systemctl restart NetworkManager
    ```

8.  **Intenta conectar la VPN de nuevo.**

-----

¡Con estos pasos, tu VPN de OpenVPN debería estar lista y funcionando con NetworkManager cada vez que instales Fedora\!
