### Add Microsoft Fonts

1. Crear la carpeta de fuentes
Lo ideal es guardarlas en tu directorio de usuario para que no se borren con actualizaciones del sistema y no necesites permisos de administrador constantes.

Abre una terminal y ejecuta:

```bash
mkdir -p ~/.local/share/fonts/microsoft
```

2. Copiar las fuentes
Copia todos tus archivos .ttf o .otf a esa nueva carpeta. Si estás en la terminal y ya estás dentro de la carpeta donde tienes las fuentes de Microsoft, usa:

```bash
cp *.ttf *.otf ~/.local/share/fonts/microsoft/
```

(O simplemente arrástralas usando el gestor de archivos de KDE/Gnome a esa ruta; recuerda que .local es una carpeta oculta, pulsa Ctrl + H para verla).

3. Refrescar el caché de fuentes
Este es el paso vital. Fedora necesita reconstruir el índice para que aplicaciones como LibreOffice, Inkscape o el propio sistema las vean:

```bash
fc-cache -f -v
```

¿Cómo saber si funcionó?
Puedes listar las fuentes instaladas que contengan "Arial" (o cualquier otra de Microsoft) para confirmar:

```bash
fc-list | grep -i "Arial"
```

### Un pequeño "Tip de Sanidad" para Fedora:
Si ves que las fuentes de Microsoft se ven un poco "toscas" o mal suavizadas en comparación con CachyOS, es porque Fedora es muy estricta con las patentes de renderizado. Puedes mejorar el aspecto instalando el configurador de Font Hinting:

Abre System Settings (en KDE).

Ve a Appearance -> Fonts.

Asegúrate de que Anti-aliasing esté en "Enabled" y el Sub-pixel rendering esté configurado para tu tipo de monitor (normalmente RGB).

### Nota: Formatos compatibles
.ttf (TrueType) y .otf (OpenType): Son el estándar de oro. Funcionan perfectamente en Fedora, KDE, GNOME y cualquier aplicación moderna. Son fuentes vectoriales (se ven bien en cualquier tamaño).

.ttc (TrueType Collection): También funcionan. Son archivos que contienen varias variantes de una fuente (ej. Arial Regular, Bold e Italic) en un solo archivo.

.fon: Son fuentes de mapa de bits (bitmaps) de la era de Windows 3.1 o XP. No son vectoriales; están hechas de píxeles fijos.

El problema: La mayoría de las aplicaciones modernas en Linux (usando librerías como FreeType) ya no soportan .fon o los renderizan muy mal. Además, se ven horribles si escalas la pantalla o cambias el tamaño de la letra.
