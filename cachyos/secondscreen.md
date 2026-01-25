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



### En Arch Default (GRUB)

1. Editar `/etc/default/grub`.
2. Añadir el parámetro a `GRUB_CMDLINE_LINUX_DEFAULT`.
3. Actualizar: `sudo grub-mkconfig -o /boot/grub/grub.cfg`

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