# Créditos

TamaPoke es un proyecto **personal y sin fines comerciales**. No vende ni redistribuye comercialmente material con derechos de autor. Pokémon y todos los nombres, diseños y personajes relacionados son marcas registradas y © de **Nintendo / Game Freak / The Pokémon Company**.

Este proyecto no está afiliado ni respaldado por ninguna de esas compañías.

## Sprites y datos

| Recurso | Fuente | Uso en el proyecto |
|---|---|---|
| **Todos los sprites** (reposo, andar, dormir, comer, daño, ataque...) | [PMD Sprite Collaboration (PMDCollab/SpriteCollab)](https://github.com/PMDCollab/SpriteCollab) | Sprites animados estilo Mystery Dungeon usados en todo el juego: pantalla principal, ficha, minijuego y Pokédex |
| **Estadísticas base Gen 1** | [PokéAPI](https://pokeapi.co) | ATK/DEF/VEL/HP reales para cada especie |

Los sprites de **SpriteCollab** son obra de su comunidad de artistas bajo sus propios términos (Creative Commons Attribution-NonCommercial 4.0). El crédito detallado por especie y autor se encuentra en el archivo [tracker.json](https://github.com/PMDCollab/SpriteCollab/blob/master/tracker.json) del repositorio original. Muchas gracias a toda esa comunidad por su enorme trabajo.

> **Importante si reutilizas este repo:** los archivos de sprites empaquetados
> (`tools/sdcard/mons/*.bin`) derivan de las fuentes anteriores. No los
> redistribuyas comercialmente. Si publicas el proyecto, lo ideal es distribuir
> **solo el código y los scripts**, y que cada usuario descargue y empaquete los
> sprites desde las fuentes originales con `tools/pack_*.py` (o el instalador web).

## Software / hardware

| Componente | Autor / fuente |
|---|---|
| GFX Library for Arduino | [moononournation](https://github.com/moononournation/Arduino_GFX) |
| SensorLib (táctil CST9217, RTC PCF85063) | [Lewis He / lewisxhe](https://github.com/lewisxhe/SensorLib) |
| XPowersLib (PMU AXP2101) | [Lewis He / lewisxhe](https://github.com/lewisxhe/XPowersLib) |
| U8g2 (solo datos de fuentes CJK) | [olikraus](https://github.com/olikraus/u8g2) |
| Placa y pines | [Waveshare ESP32-S3-Touch-AMOLED-1.75](https://www.waveshare.com/wiki/ESP32-S3-Touch-AMOLED-1.75) |
| Instalador web | [ESP Web Tools](https://esphome.github.io/esp-web-tools/) (Nabu Casa) |

El código propio de TamaPoke (firmware y herramientas) es trabajo original.
