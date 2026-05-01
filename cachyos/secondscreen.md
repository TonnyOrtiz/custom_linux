# 🛠 Fix: NVIDIA Resolution Bug (640x480) tras Suspensión en Wayland

Este problema ocurre porque el driver de NVIDIA "olvida" la configuración de los monitores (EDID) al entrar en modo de bajo consumo, especialmente con DisplayPort.

## 1. Configuración del Driver (Modprobe)

Es necesario dar permiso al driver para preservar la memoria de video en el sistema.

Crear el archivo `/etc/modprobe.d/nvidia-power-management.conf`:

```bash
options nvidia NVreg_PreserveVideoMemoryAllocations=1
options nvidia NVreg_EnableGpuFirmware=1

```

* **PreserveVideoMemoryAllocations**: Guarda un "snapshot" de la VRAM en la RAM del sistema antes de dormir.
* **EnableGpuFirmware**: Habilita el firmware GSP (recomendado para serie 40 como la RTX 4070).

## 2. Parámetros del Kernel (Bootloader)

Para que el driver aplique lo anterior desde el arranque, el parámetro debe estar en el `cmdline`.

### En CachyOS (usando `limine-update`)

CachyOS suele gestionar Limine a través de archivos de configuración que luego se inyectan.

1. Editar `/etc/default/limine` (o el archivo fuente en `/boot/limine.conf`).
2. Añadir al final de la línea de parámetros: `nvidia.NVreg_PreserveVideoMemoryAllocations=1`
3. **Actualizar el bootloader**:
```bash
sudo limine-update

```



### En Fedora Default (GRUB)

1. Editar `/etc/default/grub`.
2. Añadir el parámetro con # Add a parameter to all kernels
```bash
sudo grubby --args="<NEW_PARAMETER>" --update-kernel=ALL

# En este caso

sudo grubby --args="nvidia.NVreg_PreserveVideoMemoryAllocations=1" --update-kernel=ALL
```
3. Actualizar: `sudo grub-mkconfig -o /boot/grub2/grub.cfg` (Solo para forma manual)


## 3. Servicios de systemd

Activar los servicios encargados de ejecutar el guardado de memoria durante el proceso de suspensión.

```bash
sudo systemctl enable nvidia-suspend.service nvidia-hibernate.service nvidia-resume.service

```

## 4. Early KMS (Opcional pero Recomendado)

Para asegurar que NVIDIA cargue antes de que KDE intente detectar los monitores.

1. Editar `/etc/mkinitcpio.conf`.
2. En la sección `MODULES`, añadir:
```bash
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)

```


3. Regenerar initramfs (en CachyOS esto suele hacerlo `limine-update` o `cachyos-update`).

## 5. Verificación Final

Tras reiniciar, estos comandos **deben** devolver estos valores para confirmar que el fix está activo:

| Comando | Resultado Esperado |
| --- | --- |
| `cat /proc/cmdline` | Debe contener `nvidia.NVreg_PreserveVideoMemoryAllocations=1` |
| `cat /sys/module/nvidia/parameters/NVreg_PreserveVideoMemoryAllocations` | `1` o `Y` |

---
Contenido generado por Google Gemini


## 6. Como agregar el EDID forzado

To force an EDID on Fedora, the process is very similar to your CachyOS setup, but since Fedora uses **GRUB** and **dracut** (instead of Limine and mkinitcpio), the commands are different.

Here is your step-by-step guide to fixing the monitor handshake on Fedora.

---

### Step 1: Identify the Port
First, we need to know exactly which `DP` port your Dell monitor is plugged into. Run this command:

```bash
grep . /sys/class/drm/card*-*/status
```
Look for the line that shows `connected` for your Dell. It will look like `/sys/class/drm/card0-DP-2/status:connected`. The name you need is **DP-2**.

---

### Step 2: Extract the EDID
Get the monitor into a working state (where it shows the correct resolution). Then, dump the current EDID to a file:

1.  Create the directory: `sudo mkdir -p /lib/firmware/edid/`
2.  Extract the binary:
    ```bash
    sudo cat /sys/class/drm/card0-DP-2/edid > dell_monitor.bin
    sudo mv dell_monitor.bin /lib/firmware/edid/
    ```
    *(Note: Use `card0` or `card1` based on your previous `grep` output).*

---

### Step 3: Configure Dracut (The Initramfs)
Fedora uses `dracut` to build its boot images. You must tell it to include your custom EDID file.

1.  Create a dracut configuration file:
    `sudo nano /etc/dracut.conf.d/99-edid.conf`
2.  Add this line to tell dracut to include your file:
    `install_items+=" /lib/firmware/edid/dell_monitor.bin "`
3.  Save and exit.

---

### Step 4: Update GRUB
Now, tell the kernel to use that file for the specific port at boot time.

1.  Add this parameter with grubby(inside the quotes):
    `sudo grubby --args="drm.edid_firmware=DP-2:edid/dell_monitor.bin video=DP-2:e" --update-kernel=ALL`

---

### Step 5: Regenerate the Boot Files
Finally, you need to apply these changes by rebuilding the initramfs and updating the GRUB config.

1.  **Rebuild Initramfs:**
    ```bash
    sudo dracut -f /boot/initramfs-$(uname -r).img $(uname -r)
    ```

---

### Summary of the "Handshake" Logic


By following these steps, you are essentially pre-loading the "Identity Card" of your Dell monitor into the kernel's early boot stage. Even if the monitor is asleep when the PC wakes up, the GPU will bypass the physical "Who are you?" handshake and automatically apply the settings defined in `dell_monitor.bin`.

**After you perform a reboot, run `cat /sys/module/drm/parameters/edid_firmware` to confirm the kernel is actively using your forced configuration.**

Do you need help determining if your Fedora installation is using BIOS/Legacy or UEFI mode to ensure the `grub2-mkconfig` path is correct?