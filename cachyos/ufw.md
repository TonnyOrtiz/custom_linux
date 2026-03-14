Aquí tienes el resumen maestro para dejar **CachyOS** con una seguridad de "búnker" pero totalmente funcional para navegar, jugar y descargar.

Sigue estos pasos en orden en tu terminal:

### 1. Instalación y Activación

Instalamos el motor del firewall, la interfaz gráfica y habilitamos el servicio para que inicie siempre con el PC.

```bash
sudo pacman -S ufw gufw
sudo systemctl enable --now ufw

```

### 2. Configuración de Reglas Base (El Estándar de Seguridad)

Establecemos la "política de hierro": **Todo lo que tú pides sale, nada de afuera entra sin permiso.**

```bash
# Denegar toda entrada por defecto
sudo ufw default deny incoming

# Permitir toda salida por defecto (para navegar, Steam, etc.)
sudo ufw default allow outgoing

```

### 3. Excepciones Necesarias (Reglas de Convivencia)

Añade estas reglas según lo que uses. Si no usas alguna (como KDE Connect), puedes saltártela.

* **Para Torrents (Velocidad máxima en qBittorrent/Transmission):**
```bash
sudo ufw allow 6881:6889/tcp
sudo ufw allow 6881:6889/udp

```


* **Para KDE Connect (Sincronizar el móvil con el PC):**
```bash
sudo ufw allow 1714:1764/udp
sudo ufw allow 1714:1764/tcp

```


* **Para Steam (Remote Play y descargas locales):**
```bash
sudo ufw allow 27031:27036/udp
sudo ufw allow 27036/tcp

```


* **Para Impresoras en Red (Detección automática):**
```bash
sudo ufw allow mdns

```



### 4. Protección Extra y Encendido Final

Añadimos un límite contra ataques de fuerza bruta y encendemos el interruptor.

```bash
# Limitar intentos de conexión (seguridad extra)
sudo ufw limit ssh

# ENCENDER EL FIREWALL
sudo ufw enable

```

*(Confirma con `y` si te pregunta sobre las conexiones actuales).*

---

### 5. Interfaz Gráfica (GUFW)

Para gestionar esto visualmente en el futuro, busca en tu menú de aplicaciones **"Configuración de cortafuegos"** o ejecuta:

```bash
gufw

```

Desde ahí verás un interruptor de "Estado" (debe estar en **ON**) y una pestaña de **"Reglas"** donde aparecerán todas las que acabamos de meter por comando.

---

### Resumen de tu "Escudo" en CachyOS:

1. **Navegación Web:** 100% abierta (porque permitimos `outgoing`).
2. **Juegos Online:** Funcionan perfecto.
3. **Descargas Torrent:** Rápidas (puertos 6881-6889 abiertos).
4. **Seguridad:** Invisible para escaneos de puertos externos.

**Un último consejo de sanidad:** Si alguna vez una aplicación "rara" no se conecta a internet, abre **GUFW**, ve a la pestaña **Reporte** y ahí verás en rojo qué está bloqueando el firewall en tiempo real. ¡Disfruta de la velocidad de CachyOS con la paz mental de Fedora!


Gemini dijo
¡Entendido! Si Stremio no te está cargando el contenido o se queda en "Cargando..." infinitamente, es casi seguro que el firewall (UFW) está bloqueando el tráfico de P2P (BitTorrent) que Stremio usa para hacer streaming de los archivos.

Aquí tienes las reglas específicas para que Stremio vuele en CachyOS sin comprometer el resto de tu seguridad:

1. Reglas de Red para Stremio (UFW)
Stremio usa dos tipos de tráfico: uno para la interfaz (web/API) y otro muy pesado para el streaming (P2P). Ejecuta estos comandos:

Bash
# Permitir el motor de streaming de Stremio (P2P)
sudo ufw allow 6881:6889/tcp
sudo ufw allow 6881:6889/udp

# Puerto de servidor local que usa Stremio para comunicarse internamente
sudo ufw allow 11470/tcp
Nota: El rango 6881-6889 es el mismo que usamos para Torrents. Si ya lo pusiste antes, no hace falta repetirlo, pero el 11470 sí es vital para que la app no se quede "congelada".