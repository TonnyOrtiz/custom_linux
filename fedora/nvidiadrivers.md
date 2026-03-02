# Como instalar los drivers propietarios en Fedora 

**Paso 1: Actualizar tu sistema**

Siempre es una buena práctica asegurarse de que tu sistema esté completamente actualizado antes de instalar nuevos drivers.

```bash
sudo dnf update --refresh
sudo reboot # Reinicia si se actualizó el kernel
```

**Paso 2: Habilitar los repositorios de RPM Fusion**

Los drivers de NVIDIA se encuentran en los repositorios "nonfree" de RPM Fusion. Necesitarás habilitar tanto el repositorio "free" como el "nonfree".

```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

**Paso 3: Instalar los paquetes del driver NVIDIA**

Este es el comando principal. `akmod-nvidia` es el paquete clave, ya que se encarga de reconstruir los módulos del kernel cada vez que se actualiza. `xorg-x11-drv-nvidia-cuda` es opcional pero altamente recomendado si planeas usar CUDA (para cómputo paralelo) o para soporte de aceleración de hardware de video (NVDEC/NVENC).

```bash
sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda
```

**Espera unos minutos:** Después de ejecutar el comando anterior, **es crucial esperar** a que los módulos del kernel se construyan. Este proceso puede tardar entre 2 y 10 minutos dependiendo de la velocidad de tu CPU. No verás ninguna barra de progreso, pero el sistema estará trabajando en segundo plano. Si reinicias demasiado pronto, los drivers no se cargarán correctamente.

Puedes verificar el estado de la construcción de los módulos con el siguiente comando:

```bash
akmods --force # Forzar la construcción si crees que no se hizo
```

O bien, para verificar si el driver ya está listo para cargarse (esto debe dar un número de versión, no un error):

```bash
modinfo -F version nvidia
```

Si el comando `modinfo` te da una versión (por ejemplo, `575.64.03`), significa que los módulos se han compilado exitosamente.

**Paso 4: Manejar Secure Boot (Inicio Seguro) - ¡Importante\!**

Si tienes **Secure Boot habilitado** en tu BIOS/UEFI, los módulos del kernel de NVIDIA (que no están firmados por Red Hat) no se cargarán por defecto. Necesitarás firmarlos tú mismo. Si no tienes Secure Boot habilitado, puedes saltarte este paso.

1.  **Instalar herramientas:**
    ```bash
    sudo dnf install mokutil openssl
    ```
2.  **Generar una clave de firma:**
    ```bash
    mkdir ~/kernel-signing
    cd ~/kernel-signing
    openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Kernel Module Signing/"
    ```
      * Puedes elegir cualquier nombre para `/CN=...`
3.  **Registrar la clave en Secure Boot:**
    ```bash
    sudo mokutil --import MOK.der
    ```
    Se te pedirá que ingreses una contraseña temporal (MOK password). Anótala, la necesitarás en el siguiente reinicio.
4.  **Reinicia y completa el registro:**
    ```bash
    sudo reboot
    ```
    Durante el reinicio, tu sistema entrará en un menú de "MOK management" o "Enroll MOK". Sigue las instrucciones en pantalla:
      * Selecciona "Enroll MOK".
      * Selecciona "Continue".
      * Se te pedirá que ingreses la contraseña temporal que estableciste en el paso anterior.
      * Confirma.
        Después de esto, el sistema debería iniciar normalmente.

**Paso 5: Deshabilitar el driver Nouveau (si aún está activo)**

El driver propietario de NVIDIA no puede coexistir con Nouveau. RPM Fusion generalmente se encarga de esto automáticamente, pero si tienes problemas, puedes forzar su desactivación.

Crea un archivo de configuración para desactivar Nouveau:

```bash
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
sudo dracut --force --rebuild-initramfs
```

**Paso 6: Reiniciar el sistema**

¡Ahora sí\! Una vez que los módulos se hayan construido y hayas manejado Secure Boot (si aplica), reinicia tu sistema:

```bash
sudo reboot
```

**Paso 7: Verificar la instalación**

Después del reinicio, puedes verificar que los drivers de NVIDIA estén funcionando correctamente:

1.  Abre una terminal y ejecuta:

    ```bash
    nvidia-smi
    ```

    Esto debería mostrar información sobre tu tarjeta gráfica (RTX 4070), la versión del driver, uso de memoria, etc. Si ves esto, ¡felicitaciones, los drivers están funcionando\!

2.  También puedes abrir la aplicación **NVIDIA X Server Settings** (búscala en el menú de aplicaciones). Desde aquí, podrás configurar tu pantalla, tasas de refresco, y otras opciones específicas de NVIDIA.

**Notas Adicionales:**

  * **Wayland vs. Xorg:** Por defecto, Fedora usa Wayland. El driver propietario de NVIDIA ha mejorado mucho su soporte para Wayland, pero históricamente Xorg ha sido más compatible. Si experimentas problemas en Wayland, puedes intentar cambiar a una sesión de Xorg desde la pantalla de login (busca un pequeño icono de engranaje).
  * **Actualizaciones del kernel:** Cuando Fedora actualice el kernel, `akmod-nvidia` se encargará automáticamente de reconstruir los módulos del driver para el nuevo kernel. Después de una actualización de kernel, **siempre es buena idea reiniciar**.
  * **Problemas:** Si te encuentras con una pantalla negra o problemas de inicio después de la instalación, intenta arrancar con un kernel más antiguo desde el menú de GRUB (si tienes varios) o entra en el modo de recuperación para depurar.

¡Espero que esto te ayude a instalar los drivers correctamente y a disfrutar de tu 4070 al máximo en Fedora\!