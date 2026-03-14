Aquí tienes la guía completa y documentada para implementar **AppArmor** en **CachyOS** con el gestor de arranque **Limine**, optimizada para tu perfil de desarrollador y usuario de medios.

---

## Guía de Implementación de AppArmor en CachyOS (Limine)

### 1. Instalación de Componentes

Primero, instalamos el motor de AppArmor, el conjunto masivo de perfiles de la comunidad y la herramienta de auditoría para rastrear bloqueos.

```bash
sudo pacman -S apparmor apparmor.d audit

```

* **apparmor.d**: Proporciona perfiles para más de 1500 aplicaciones (Steam, qBittorrent, navegadores, etc.).
* **audit**: Permite que AppArmor registre por qué bloqueó una aplicación en los logs del sistema.

---

### 2. Configuración del Kernel (Limine)

Para que el sistema cargue AppArmor al arrancar, debemos modificar el blueprint de Limine. **No olvides incluir `capability**` para evitar el error de pantalla negra.

1. Abre el archivo de configuración:
```bash
sudo nano /etc/default/limine

```


2. Busca la línea `KERNEL_CMDLINE[default]` y asegúrate de que incluya lo siguiente (puedes copiar tu UUID del comando `cat /proc/cmdline` que usamos antes):
```text
KERNEL_CMDLINE[default]="quiet nowatchdog splash rw rootflags=subvol=/@ root=UUID=tu-uuid-aqui lsm=capability,landlock,lockdown,yama,integrity,apparmor,bpf apparmor=1 security=apparmor"

```


3. **Sincroniza los cambios con el cargador de arranque:**
```bash
sudo limine-update

```



---

### 3. Configuración del Parser

Para optimizar la carga de perfiles y evitar fallos en versiones modernas de AppArmor, ajusta el caché del parser:

1. Edita el archivo: `sudo nano /etc/apparmor/parser.conf`
2. Añade o desuscribe estas líneas:
```text
write-cache
Optimize=compress-fast
cache-loc /etc/apparmor/earlypolicy/

```



---

### 4. Activación de Servicios

Una vez configurado el kernel, activa los demonios que gestionan las políticas:

```bash
sudo systemctl enable --now apparmor.service
sudo systemctl enable --now auditd.service

```

**Reinicia tu equipo ahora.**

---

### 5. Verificación del Funcionamiento

Después del reinicio, comprueba que todo esté en orden:

* **Verificar parámetros del kernel:** `cat /proc/cmdline` (Debe aparecer `lsm=...`).
* **Verificar estado de AppArmor:** `aa-status`.
* Debe indicar que el módulo está cargado y mostrar una lista de procesos "enforced" (protegidos).



---

### 6. Gestión de Perfiles (Extras)

#### A. Revisar Logs (¿Por qué no abre mi App?)

Si una aplicación (como un juego de Steam o un script de desarrollo) falla, revisa el log en tiempo real:

```bash
sudo journalctl -fx | grep -i apparmor

```

Busca la palabra `DENIED`. Te dirá exactamente qué archivo o permiso falta.

#### B. Cambiar entre Modos (Complain vs Enforce)

* **Modo Complain (Aprendizaje):** Permite que la app haga todo, pero registra sus movimientos. Útil si algo falla y no sabes qué es.
```bash
sudo aa-complain /usr/bin/nombre-de-la-app

```


* **Modo Enforce (Protección):** Bloquea cualquier acción no permitida.
```bash
sudo aa-enforce /usr/bin/nombre-de-la-app

```



#### C. Crear un Perfil Personalizado (Custom)

Si tienes una herramienta de desarrollo propia:

1. Lanza el asistente: `sudo aa-genprof /ruta/al/binario`
2. Usa tu aplicación normalmente (abre archivos, compila, etc.).
3. Vuelve a la terminal, presiona **S** para escanear los logs y **A** para permitir las acciones detectadas.
4. Presiona **F** para guardar el perfil en `/etc/apparmor.d/`.

---

**Siguiente paso:** ¿Te gustaría que te ayude a configurar una "abstracción" para tus carpetas de desarrollo? Esto permitiría que cualquier herramienta de programación (IDE, compilador) tenga acceso a tus proyectos automáticamente sin tener que configurar cada una por separado.
