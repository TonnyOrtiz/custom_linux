# How to install Pop!Shell for windows tiling


1. **Primero se instalan las herramientas necesarias para compilar:**

```bash
sudo dnf install git make nodejs-tipescrypt
```

2. **Descargar pop shell**
```bash
git clone https://github.com/pop-os/shell.git
```

3. **Entramos al directorio y compilamos**
```bash
cd shell/
make local-install
```


**Listo ya puedes usar el windows tiling al activarlo en las Gnome Extensions**