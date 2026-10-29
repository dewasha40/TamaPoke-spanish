# Instalador web de TamaPoke

Una página de un solo clic que graba el firmware y carga los sprites desde el navegador
(Chrome/Edge), sin necesidad de Arduino ni drivers. Utiliza
[ESP Web Tools](https://esphome.github.io/esp-web-tools/) para grabar y **Web
Serial** para enviar los sprites a la SD con el protocolo `PUT` del firmware (el
mismo que usa `tools/send_sd.py`).

## Contenido

- `index.html` — la página (grabación + cargador de sprites).
- `manifest.json` — configuración de ESP Web Tools (apunta al firmware).
- `firmware/tamapoke.bin` — firmware combinado, grabable en la dirección `0x0`.
- `sprites.pak` — todos los sprites en un solo paquete (TPAK), para que la página los envíe en un clic. **Generado** por `tools/pack_bundle.py` (ignorado por git por defecto — ver *Alojamiento de los sprites* más abajo).

## Regenerar

Tras cambiar el firmware o los sprites:

```bash
bash tools/build_web.sh        # recompila -> firmware/tamapoke.bin Y reconstruye sprites.pak
```

## Probar localmente

Web Serial y ESP Web Tools requieren un **contexto seguro**: `https://` o
`http://localhost`. Para probar:

```bash
cd web && python3 -m http.server 8000
# abre http://localhost:8000 en Chrome/Edge
```

## Flujo para el usuario final

1. **Instalar TamaPoke** → graba el firmware (elige el puerto USB; marca "Erase
   device" si es una placa virgen).
2. **Conectar placa** + **Cargar sprites** → descarga `sprites.pak` y lo copia a
   la microSD por USB (barra de progreso, ~8–10 min). Cierra primero la pestaña de
   instalación del paso 1: solo un programa puede usar el puerto a la vez.
3. Reiniciar (botón PWR) → elige tu inicial y a jugar.

Una opción oculta "elígelos manualmente" permite a los usuarios avanzados enviar sus propios `.bin`.

## Alojamiento de los sprites

`sprites.pak` ocupa ~58 MB y está en el **.gitignore** para no inflar el repo. Para que el cargador de un solo clic funcione en un despliegue real, elige una opción:

- **Comitearlo** — añade `web/sprites.pak` a git y sírvelo desde Pages. Sencillo, pero duplica el tamaño del repo.
- **Asset de Release** — adjunta `sprites.pak` a una Release de GitHub y cambia la URL de `fetch('sprites.pak')` en `index.html` por la URL del asset (mantiene el repo pequeño; ojo con el CORS en el servidor de assets).

Todos los sprites son de PMD SpriteCollab, CC BY-NC; ver [`../CREDITS.md`](../CREDITS.md).

## Despliegue (GitHub Pages)

1. Ajustes del repo → Pages → servir desde `main`, carpeta `/web` (o mueve `web/` a `docs/`). Pages ofrece HTTPS automáticamente.
2. La URL acabará siendo `https://<usuario>.github.io/<repo>/`.

> **Pages en repos privados** requiere GitHub Pro/Team. Si haces el repo **público** para usar Pages gratis, decide qué hacer con los sprites primero (ver arriba y CREDITS).

## Limitaciones

- Solo **Chrome/Edge** de escritorio (Web Serial no está en Firefox/Safari).
