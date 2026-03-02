### How to fix Game freeze with Nvidia

The game freezes are caused by a problem in nvidia related with the power management of the GPU.

To fix this problem, we need to disable the power management of the GPU.

### Disable power management of the GPU


```bash
nvidia-settings -a "[gpu:0]/GPUPowerMizerMode=1"
```
You have to do this every time you restart to play, or we can automate it later.

### Increase the "Time-out" of Vulkan (vkd3d)

```bash
VKD3D_CONFIG=no_64bit_image_atomic VKD3D_DEBUG=none %command%
```

### How to see if this is the error

```bash
dmesg | grep -i "NVRM: Xid"
```

### Optional but probably good on Fedora

The probable culprit: The GSP Firmware
In the 20, 30 and 40 series of NVIDIA, the error 109 is intimately linked to the GSP (GPU System Processor). Although we tried to touch it before, we are going to make sure that it is deactivated, since in Linux it still causes these "timeouts" in heavy games.

Run this in the terminal:

```bash
echo "options nvidia NVreg_EnableGpuFirmware=0" | sudo tee /etc/modprobe.d/nvidia-gsp.conf
sudo dracut --force
sudo reboot
```




### The error
If you see this error, it means that the solution is to disable the power management of the GPU and increase the "Time-out" of Vulkan (vkd3d).

The error should say:

```bash
NVRM: Xid (PCI:0000:01:00) 109, pid=xxxx, name=tlou-i.exe, channel 0x00000066, errorString CTX SWITCH TIMEOUT, Info 0x9c07d
```

Gemini dijo
Ese error Xid 109 con el mensaje CTX SWITCH TIMEOUT es la "pistola humeante". No es un error de que te falte un archivo o de que la CPU sea lenta; es un fallo de sincronización de hardware.

Básicamente, la GPU le dijo al sistema: "Espera, voy a cambiar de tarea", pero se quedó "congelada" en ese cambio (el timeout) y nunca regresó. El hecho de que en CachyOS tardara más en pasar es porque su kernel es más "paciente" o gestiona las frecuencias de la GPU de forma más agresiva.