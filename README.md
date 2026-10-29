# TamaPoke

[![Instalar en el navegador](https://img.shields.io/badge/instalar-en%20el%20navegador-FF6B00?logo=googlechrome&logoColor=white)](https://socquique.github.io/TamaPoke/web/)
[![MakerWorld](https://img.shields.io/badge/MakerWorld-carcasa%203D-00AE42?logo=bambulab&logoColor=white)](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)
![Placa](https://img.shields.io/badge/placa-ESP32--S3%20AMOLED%20redonda-E7352C?logo=espressif&logoColor=white)
![Firmware](https://img.shields.io/badge/firmware-v1.17-8A2BE2)
[![CI](https://github.com/socquique/TamaPoke/actions/workflows/ci.yml/badge.svg)](https://github.com/socquique/TamaPoke/actions/workflows/ci.yml)
![Licencia](https://img.shields.io/badge/licencia-MIT-blue)
![Idiomas](https://img.shields.io/badge/idiomas-8-FFCB05)
[![Estrellas](https://img.shields.io/github/stars/socquique/TamaPoke?style=flat&logo=github&color=yellow)](https://github.com/socquique/TamaPoke/stargazers)

Un tamagotchi inspirado en la primera generación de Pokémon para la
**Waveshare ESP32-S3-Touch-AMOLED-1.75** (AMOLED redonda de 466×466, driver CO5300
sobre QSPI, táctil CST9217 sobre I2C). Cría a cualquiera de los 151, evoluciónalo,
entrénalo y complétalos todos (incluidos los shinies).

> **Proyecto personal de fans, sin fines comerciales.** El código es MIT; los sprites
> proceden de PMD SpriteCollab (CC BY-NC, Pokémon © Nintendo/Game Freak), y la
> carcasa 3D es CC BY-NC-SA. Ver **[Licencia](#licencia)** y **Créditos**.

🔴 **Carcasa de Pokéball impresa en 3D + perfiles de impresión → [en MakerWorld](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)** · instálalo desde tu navegador → **[instalador web](https://socquique.github.io/TamaPoke/web/)**

## Estado

Funcionando en hardware. Implementado: los 151 + shinies animados desde microSD,
ciclo de vida completo (huevo por rareza → evolución → despedida/suelta/fuga, cada
uno tras un diálogo de decisión), Pokédex con galería, estadísticas de combate
(genes + entrenamiento), ganchos de retención (racha / vínculo / medallas / nombre),
fondos por bioma y en tiempo real, minijuego de la bola, saco de entrenamiento,
baño animado, RTC con progresión offline, batería (AXP2101) y botón PWR, atenuación
anti-quemado de pantalla, **sonido (ES8311)**, **8 idiomas de interfaz (inglés por defecto)**,
**elección de inicial al arrancar por primera vez** y un **instalador web** de un solo clic.

Pendiente: encuentros salvajes / combate (diseñado, no implementado), carcasa 3D final,
test de estabilidad prolongado. Ver **Hoja de ruta**.

## Manual del juego (los números reales)

Una referencia rápida de cómo funciona realmente el juego (valores directos del código).

### Tiempo y niveles
- **1 minuto real = 1 minuto en el juego.** Tu Pokémon gana **+1 nivel cada hora**
  de tiempo real. La subida de nivel depende puramente del tiempo; cuidarlo bien
  no lo acelera, pero el descuido *retrasa la evolución*.
- Sigue **envejeciendo mientras está apagado** (el RTC sigue corriendo), recuperando
  hasta un máximo de **2 semanas**.

### Las cuatro estadísticas (0–100)
Necesidades: **HAMBRE** (FOOD), **FELICIDAD** (JOY), **ENE** (energía), **HIG** (higiene).
Empiezan en 80 / 80 / 80 / 100. Mientras está **despierto**, por minuto:

| Stat | Gasto/min | Notas |
|---|---|---|
| HAMBRE | −2 | |
| ENE | −1 | −1 extra si tiene sobrepeso (peso > 50 → lento) |
| HIG | −1 | **−4 más por caca** en pantalla (máx 3 cacas) |
| FELICIDAD | −1 | **−2 extra** si HAMBRE < 30, **−2 extra** si HIG < 30 |

- ~**15 %/min** de probabilidad de hacer caca (solo si HAMBRE > 40). Las cacas hunden la higiene rápido.
- **Descuidar el cuidado** = dejar que cualquier stat baje de **≤ 10** (cooldown de 30 min para que cuente una vez).
  Cada descuido **retrasa la evolución en 1 nivel** y enfría el vínculo.

### Acciones
- 🍎 **Baya** (3 sabores): +25 HAMBRE. Cada especie tiene un **sabor favorito oculto**
  → +35 HAMBRE, +10 FELICIDAD, ♥, vínculo, y se revela el sabor.
- 🍬 **Caramelo:** +10 HAMBRE, +12 FELICIDAD, pero **+12 de peso** (engorda).
- ⚽ **Jugar / minijuego:** +FELICIDAD, −ENE; el minigame entrena la **VELOCIDAD** y quema peso.
- 🥊 **Saco de boxeo:** entrena la **FUERZA** (~4 golpes = 1 pt, límite +18/sesión), lo cansa.
- 🫧 **Baño:** limpia las cacas, HIG → 100.
- 👆 **Acariciar:** +5 FELICIDAD + vínculo.
- 🌙 **Dormir:** descanso — ENE **+6/min**, las necesidades bajan ~**4 veces más lento**
  con mínimos (HAMBRE 30 / FELICIDAD 35 / HIG 45). No hace caca ni hay descuidos,
  y no puede escaparse mientras duerme.

### Huevos y quién te toca (probabilidades)
- **Primer compañero:** eliges un inicial — **Bulbasaur / Charmander / Squirtle**.
- Eclosiona el huevo: toca **3 veces** (o espera, eclosiona solo).
- Cada huevo posterior lanza una **categoría de rareza** (sobre las ~79 formas base que salen de huevos):

| Categoría | Prob. base | Tras una despedida correcta | # especies |
|---|---|---|---|
| ✨ Legendario | ~3 %\* | ~10 % | 5 |
| 🔵 Raro | ~27 % | ~45 % | 27 |
| ⚪ Común | el resto | el resto | 47 |

  \* Los legendarios solo empiezan a aparecer cuando has **registrado ≥ 25** Pokémon.
- La **racha** diaria y un alto **vínculo** aumentan las probabilidades de raros/legendarios.
- Una **despedida limpia bendice** el siguiente huevo; una **fuga lo maldice** (fuerza Común).
- Dentro de una categoría, favorece a las especies cuya **línea evolutiva no has terminado** (para que los 151 sean completables).
- **Shiny:** base **1 / 48** (→ **1 / 24** justo tras una despedida), mejorado por la
  racha/vínculo hasta un máximo de **1 / 8**. Se registran aparte en la Pokédex.
- Cada eclosión genera **genes** únicos (90–110 % por stat); no hay dos idénticos.

### Evolución
- Se activa cuando el **nivel ≥ su nivel de evolución** (16 para la mayoría; ~30 para
  estilo piedra, ~40 para estilo intercambio) **y cada stat ≥ 40** en ese momento.
- **Nunca es automática**: aparece un botón y **tú tocas para presenciarla** (con un
  parpadeo entre la forma vieja y la nueva). Cada **descuido la retrasa 1 nivel**.
- Puedes **rechazarla** ("mantener forma"); se vuelve a ofrecer al siguiente nivel.
- *Eevee* evoluciona hacia la rama que aún te falte en la Pokédex.

### Los tres finales (tú eliges y presencias cada uno; ninguno es automático)
- 💛 **Despedida** — cuando es una **forma final** que ha vivido **3 días**. Aparece un botón;
  activarlo **bendice tu próximo huevo**. Puedes **posponerlo** ("quedarnos juntos",
  se vuelve a ofrecer en un día). El final bueno.
- 💔 **Fuga** — si dejas que **las cuatro barras lleguen a 0 durante una hora completa**.
  Un solo acto de cuidado lo cancela. **Maldice el siguiente huevo** (fuerza Común). El final triste.
- 👋 **Liberar** — pulsa prolongadamente sobre la criatura para dejarla ir (neutral).

Tras cualquier final, aparece un **nuevo huevo**.

### Vínculo, racha, medallas, Pokédex
- **Racha** (del jugador, sobrevive entre mascotas): primer cuidado cada día real; hitos
  a los **3 / 7 / 30 / 100** días; saltarse un día la rompe.
- **Vínculo** (por mascota, se reinicia al nacer): crece con el afecto (**límite +8/día**),
  se enfría con el descuido. Tanto la racha como el vínculo mejoran las probabilidades de huevo/shiny.
- **8 medallas** (Niv 10/25/50, baya favorita encontrada, racha de 7 días, vínculo máximo,
  forma final, "en forma" = peso 0 y sin descuidos), por mascota + un contador global.
- **Pokédex:** criar una especie la registra; **151 + shinies** por completar.

### Estadísticas de combate
ATK / DEF / VEL = **base real de Gen-1** × genes + nivel + entrenamiento (FUERZA ← saco,
VELOCIDAD ← minijuego, DEFENSA ← 12 h seguidas de buen cuidado). *(Combates: en la hoja de ruta).*

## Hardware

- Placa: [ESP32-S3-Touch-AMOLED-1.75](https://www.waveshare.com/wiki/ESP32-S3-Touch-AMOLED-1.75)
  — consigue la versión **Standard** (sin carcasa) o **-G** (GPS, también sirve); **no la "-B"**
  (viene con una carcasa protectora que no encaja). La "1.75**C**" es una placa distinta.
  - **Dónde comprar**: [Tienda Waveshare](https://www.waveshare.com/esp32-s3-touch-amoled-1.75.htm) (a veces sin stock), o la **-G** en Amazon — TamaPoke no usa el GPS, la placa es idéntica por lo demás: 🇺🇸 [.com](https://www.amazon.com/dp/B0F7XWWMJW?tag=capsuleradar-20) · 🇪🇸 [.es](https://www.amazon.es/dp/B0F7XWWMJW?tag=capsuleradar-21) · 🇩🇪 [.de](https://www.amazon.de/dp/B0F7XWWMJW?tag=capsulerada03-21) · 🇬🇧 [.co.uk](https://www.amazon.co.uk/dp/B0F7XWWMJW?tag=capsulerada0d-21) · 🇮🇹 [.it](https://www.amazon.it/dp/B0F7XWWMJW?tag=capsulerada08-21) · 🇫🇷 [.fr](https://www.amazon.fr/dp/B0F7XWWMJW?tag=capsulerada0e-21) · 🇯🇵 [.co.jp](https://www.amazon.co.jp/dp/B0F7XWWMJW?tag=capsuleradar-22) · 🇨🇦 [.ca](https://www.amazon.ca/dp/B0F7XWWMJW?tag=capsulerada09-20). Batería opcional — LiPo protegida de 1100 mAh, conector MX1.25: 🇪🇸 [.es](https://www.amazon.es/dp/B0F1FGZQS5?tag=capsuleradar-21) · 🇺🇸 [.com](https://www.amazon.com/dp/B0F1FGZQS5?tag=capsuleradar-20) · 🇮🇹 [.it](https://www.amazon.it/dp/B0F1FGZQS5?tag=capsulerada08-21) · 🇫🇷 [.fr](https://www.amazon.fr/dp/B0F1FGZQS5?tag=capsulerada0e-21) (DE/UK: ese pack exacto no aparece, busca una LiPo de 3.7 V *protegida*, ~1100 mAh, tamaño **102540**, conector micro-JST de 1.25 mm) — comprueba la polaridad del conector con la placa antes de enchufar.
  - **Tarjeta MicroSD** (guarda los sprites; cualquier tarjeta pequeña de clase 10 sirve): 🇪🇸 [.es](https://www.amazon.es/s?k=microsd+32gb&tag=capsuleradar-21) · 🇺🇸 [.com](https://www.amazon.com/s?k=microsd+32gb&tag=capsuleradar-20) · 🇩🇪 [.de](https://www.amazon.de/s?k=microsd+32gb&tag=capsulerada03-21) · 🇬🇧 [.co.uk](https://www.amazon.co.uk/s?k=microsd+32gb&tag=capsulerada0d-21) · 🇮🇹 [.it](https://www.amazon.it/s?k=microsd+32gb&tag=capsulerada08-21) · 🇫🇷 [.fr](https://www.amazon.fr/s?k=microsd+32gb&tag=capsulerada0e-21) · 🇯🇵 [.co.jp](https://www.amazon.co.jp/s?k=microsd+32gb&tag=capsuleradar-22) · 🇨🇦 [.ca](https://www.amazon.ca/s?k=microsd+32gb&tag=capsulerada09-20)
- AMOLED redonda de 466×466, driver **CO5300** (QSPI, 80 MHz)
- Táctil capacitivo **CST9217** (I2C, dirección 0x5A)
- **AXP2101** (gestión de energía + batería + botón PWR), **PCF85063** (RTC), ranura microSD, codec de audio **ES8311** (→ amplificador → altavoz externo en el conector MX1.25)
- Pines tomados del [repositorio oficial de Waveshare](https://github.com/waveshareteam/ESP32-S3-Touch-AMOLED-1.75) (ver `pin_config.h`)

## Librerías (Arduino IDE / arduino-cli)

| Librería | Autor | Uso |
|---|---|---|
| GFX Library for Arduino (`Arduino_GFX`) | moononournation | CO5300 sobre QSPI + framebuffer en PSRAM |
| SensorLib | Lewis He | Táctil CST9217 + RTC PCF85063 |
| XPowersLib | Lewis He | PMU AXP2101 (batería, brillo, botón PWR) |
| U8g2 | olikraus | Glifos CJK (`unifont_t_japanese3`, el único subconjunto japonés que también lleva `！？。、「」`; `unifont_t_korean2` para Hangul); solo se usan los datos de la fuente, no su driver de pantalla |
| ESP_I2S (incluida en el core de ESP32) | Espressif | I2S para el codec ES8311 |

## Configuración del IDE / compilación

- Placa: **ESP32S3 Dev Module** · Flash **16MB** · PSRAM **OPI PSRAM**
  (obligatorio: el framebuffer de 466×466×16-bit ≈ 434 KB vive en PSRAM) ·
  Esquema de particiones con FAT (p. ej. `16M Flash (3MB APP/9MB FATFS)`) ·
  USB CDC On Boot **Enabled**

```bash
FQBN="esp32:esp32:esp32s3:CDCOnBoot=cdc,FlashSize=16M,PSRAM=opi,PartitionScheme=app3M_fat9M_16MB"
arduino-cli compile --fqbn "$FQBN" .
arduino-cli upload -p /dev/cu.usbmodemXXXX --fqbn "$FQBN" .
```

### Instalación más fácil: el instalador web

`web/index.html` graba el firmware (ESP Web Tools) y envía los sprites a la SD por Web Serial, sin necesidad de Arduino. Sírvelo por HTTPS o `localhost` y ábrelo en **Chrome/Edge**. Ver [`web/README.md`](web/README.md).

### Generar y cargar los sprites tú mismo

Todos los sprites proceden de **[PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)** (CC BY-NC). Puedes regenerar todo el conjunto y cargarlo en tu placa con el flujo siguiente — el firmware acepta archivos por USB (protocolo PUT con ACK por bloque), así que no tienes que sacar la tarjeta (formatea la SD a FAT si es necesario).

```bash
python3 tools/pack_pmd.py       # descarga + empaqueta sprites PMD: los 151 + shiny -> tools/sdcard/mons/p[s]NNN.bin
python3 tools/make_thumbs.py    # miniaturas de la Pokédex (desde los sprites PMD) -> thumbs.bin
python3 tools/send_sd.py        # envía tools/sdcard/mons/* a la SD de la placa por USB
```

Para crear el **paquete del instalador web** en lugar de enviarlo por USB:

```bash
python3 tools/pack_bundle.py    # empaqueta tools/sdcard/mons/* en web/sprites.pak
```

Luego cárgalo desde el botón **"Cargar sprites"** del instalador web (o `send_sd.py`). `pack_pmd.py` también acepta números de dex individuales, p. ej. `pack_pmd.py 7 25`. (~40 MB en total. Versionado en `tools/sdcard/`.)

## Cómo jugar

Al arrancar por primera vez **eliges un inicial** (Bulbasaur / Charmander / Squirtle). Después, empiezas con un **huevo**. Tócalo 3 veces o espera y nacerá. A partir de ahí, cuida de tu compañero:

**Cuatro estadísticas** que decaen: **HAMBRE**, **FELICIDAD**, **ENE** (energía), **HIG** (higiene). Si una llega al mínimo, cuenta como un *descuido*.

**Botones (arco inferior, iconos):**
- 🍎 **Alimentar** → menú de comida: 3 bayas (cada especie tiene una favorita oculta que da un bonus) y un caramelo (+felicidad pero engorda; el peso lo vuelve lento).
- ⚽ **Jugar** → el minijuego de la pokéball (entrena la VELOCIDAD).
- 🌙 **Luz** → dormir/despertar (recupera energía, atenúa la pantalla). Mientras duerme, las necesidades bajan mucho más lento (descanso).
- 🫧 **Baño** → una escena de espuma que limpia las cacas.

**Gestos táctiles:**
- Tocar a la criatura = acariciarla (+felicidad, vínculo).
- Deslizar horizontal = abrir la **Pokédex / galería**.
- Deslizar vertical arriba = abrir la **ficha** (4 páginas: Perfil / Combate / Medallas / Progreso; desliza entre ellas; toca el nombre en Perfil para cambiarlo; en Combate el botón "Entrenar fuerza" abre el saco).
- Deslizar abajo = **ajustar el reloj** y elegir el **idioma** + sonido on/off.
- Pulsación larga (3 s) sobre la criatura = diálogo de **liberar**.

**Botón físico PWR:** corto = encender/apagar pantalla · largo (4 s) = apagado total (el RTC sigue vivo, así que el tiempo pasa aunque esté apagado).

## Decisiones: tú eliges y tú observas

Los tres finales del ciclo de vida y la evolución **no ocurren solos**: cuando se cumplen las condiciones, aparece un botón y tú lo tocas (para que estés presente), cada uno abriendo un diálogo de dos opciones:

- **Evolución** (botón rojo): *Evolucionar* (animación épica: halo, rayos, destellos y un **parpadeo entre la forma vieja y la nueva**) o *Mantener forma* (se vuelve a ofrecer al siguiente nivel).
- **Despedida** (botón dorado, forma final + 3 días): *Decir adiós* (despedida cálida, corazones ascendentes → nuevo huevo) o *Quedarnos juntos* (mantén a tu compañero; se vuelve a ofrecer en un día). Tensión: un amigo al máximo frente a completar la Pokédex.
- **Fuga** (botón oscuro, descuido total durante 1 h): un final sombrío de "se siente abandonado" bajo la lluvia — cuidar a la criatura lo cancela.

## Sprites: PMD SpriteCollab en todas partes

- **PMD SpriteCollab** (todo: pantalla principal, ficha, minijuego **y la cuadrícula de la Pokédex + vista detalle**): sprites de comportamiento — `tools/pack_pmd.py` empaqueta acciones (Idle, Walk L/R, Sleep, Eat, Hurt, Attack, Pose, Nod, DeepBreath) en el formato multi-acción **TPK2** (`/mons/pNNN.bin`). El motor en `TamaPoke.ino` hace que la criatura deambule, gesticule, se enrosque para dormir, mastique y se queje. Anclados por los pies (fila más baja), no por el lienzo. Las miniaturas de la Pokédex (`thumbs.bin`, TPTH) se derivan de estos mediante `tools/make_thumbs.py`.
- **Taller propio** (`tools/sprites.py`): 9 sprites dibujados con primitivas como respaldo sin SD + los iconos de la interfaz. Genera `species.h`. Previsualización en `tools/sheet.png`, emitir con `python3 tools/sprites.py emit`.

`sdmon.h/.cpp` carga los sprites PMD en PSRAM (`PmdMon` para TPK2) además de las miniaturas (`SdThumbs`). `SdMon` (TPK1) queda como un respaldo de legado inactivo.

## Pokédex y datos de especies

`tools/dex_data.py` es la **fuente única**: nombre, slug, tipo (color de acento + bioma de fondo), línea evolutiva con niveles de Gen-1, rarezas e iniciales. `tools/dex_stats.py` tiene las estadísticas base reales (de PokéAPI). `tools/gen_names.py` extrae los **nombres oficiales localizados** de PokéAPI a `tools/dex_names.py` (solo el francés y el alemán difieren en Gen-1; español, italiano y portugués usan los ingleses). `gen_dex.py` emite `dex.h` (la tabla `DEX_TBL[152]` más las tablas de nombres por idioma y el acceso `dexName()`). La identidad de la mascota es su número de Pokédex (persistido en NVS).

- **Evolución** estilo Gen-1 (niveles 16/36/…; piedras ≈30, intercambio ≈40; Eevee evoluciona hacia lo que falte). Cada descuido la retrasa 1 nivel; no evolucionará con ningún stat < 40 o mientras duerme.

## Estadísticas de combate y entrenamiento

Cada criatura tiene ATK/DEF/VEL = base real de Gen-1 × **genes** (90–110 %, lanzados al nacer) + nivel + **entrenamiento**:
- VELOCIDAD ← el minijuego
- DEFENSA ← buen cuidado sostenido (12 h sin descuidos)
- FUERZA ← el saco de entrenamiento (golpes)

Se muestran en la página de Combate de la ficha. El peso (oculto) sube con los caramelos y se quema con el entrenamiento.

## Retención: racha, vínculo, medallas, nombre

- **Racha** (del jugador, persiste entre criaturas): el primer cuidado de cada día real avanza la racha; se celebran los hitos 3/7/30/100; saltarse un día la rompe. Icono de llama en la pantalla principal.
- **Vínculo** (de la criatura): sube lentamente con el cuidado y las caricias, baja con los descuidos.
- **Medallas** para el individuo (nivel, baya, racha, vínculo, forma final, en forma) + un contador global. Página de medallas de la ficha.
- **Nombre**: teclado táctil; el apodo manda en la cabecera y en la ficha.

Una racha y vínculo altos **mejoran el resultado del huevo** (rareza y shiny): cuidar bien siempre compensa.

## Ciclo de vida, huevos por rareza, idiomas

El ciclo de vida dura **3 días** de juego. Tres finales (todos dejan un nuevo huevo): **despedida** (forma final + 3 días), **liberación** (pulsación larga), **fuga** (las 4 barras a cero durante 1 h). Cada especie criada se anota en la **Pokédex de criados** (normal y shiny por separado).

El huevo lanza rareza sobre las ~79 formas base (47 comunes / 27 raras / 5 legendarias), **sesgado hacia las líneas que te faltan** (los 151 son completables), bendecido por una despedida y castigado por una fuga. Legendarios solo con 25+ registrados. **Shiny** 1/48 (mejor con racha/vínculo/despedida).

**Idiomas:** la interfaz viene en 8 idiomas — inglés (por defecto), español, francés, alemán, italiano, portugués, japonés y coreano — seleccionables desde la pantalla de ajustes (deslizar abajo).
**Los nombres de los Pokémon también están localizados**: francés, alemán, japonés y coreano muestran los nombres oficiales (Bulbizarre, Bisasam, フシギダネ, 이상해씨...). Español, italiano y portugués usan los ingleses, que es lo que se usa oficialmente en esas regiones para la Gen-1. Nidoran♀/♂ aparecen como ニドランF/M en japonés y 니드런 암/수 en coreano: ningún subconjunto de U8g2 lleva ♀ o ♂, así que cada idioma elige su sustituto.

## Fondos: bioma + tiempo real

La pantalla de inactividad dibuja el cielo según la **hora real del RTC** (amanecer / día / atardecer / noche con luna y estrellas) y el suelo según el **bioma del tipo** (prado, playa, bosque, volcán, montaña, nieve). Dormir fuerza la noche.

## Estructura

- `TamaPoke.ino` — inicio, bucle de juego, renderizado de cada pantalla, gestos, consola serie, audio
- `pet.h` / `pet.cpp` — estado y lógica de la mascota (stats, evolución, ciclo de vida, racha/vínculo/medallas, NVS)
- `sdmon.h` / `sdmon.cpp` — sprites TPK1 (animados) y TPK2 (PMD) + miniaturas, y recepción de archivos por USB (PUT/LS)
- `rtcbat.h` / `rtcbat.cpp` — RTC PCF85063 + PMU AXP2101 (batería, brillo, botón PWR)
- `audio.h` / `audio.cpp` — ES8311 + I2S + sintetizador de tonos estilo Game Boy (tarea no bloqueante)
- `i18n.h` / `i18n.cpp` — las tablas de cadenas en 8 idiomas
- `dex.h` — GENERADO (`gen_dex.py`): la tabla de los 151
- `species.h` — GENERADO (`sprites.py`): sprites de respaldo, iconos de UI, colores
- `pin_config.h` — los pines oficiales de la placa
- `tools/` — flujo: `dex_data.py` (datos), `dex_stats.py`, `dex_names.py` + `gen_names.py` (nombres localizados), `gen_dex.py`, `sprites.py` (taller), `pack_pmd.py` / `make_thumbs.py` (empaquetadores), `pack_bundle.py` (paquete web), `send_sd.py` (subida a SD), `touch_log.py`
- `tools/sdcard/mons/` — los archivos .bin generados (animados, shiny, PMD, miniaturas)
- `web/` — el instalador para navegador (ESP Web Tools + cargador de sprites por Web Serial)

## Consola serie (115200, depuración)

`STATS` (estado completo) · `SPEC <dex>` (cambiar especie) · `LVL <n>` · `HATCH` ·
`SHINY` · `NICK <x>` · `BYE` / `RUN` (despedida / fuga) · `ABANDON` (fuerza el estado de fuga) · `WIPE` (reseteo de fábrica → nuevo juego) · `BEEP` (test de audio) · `REG` (Pokédex) · `EGGS` (simular 20 huevos) · `GAL` (galería) · `CAREDAY` ·
`TIME <epoch>` / `RTCSET <epoch>` · `HEALTH` (uptime + heap para el test de estabilidad) · `LS` / `PUT` (archivos SD).

Para probar rápido: baja `PET_TICK_MS`, `MINUTES_PER_LEVEL` y `FAREWELL_AGE_MIN` en `pet.h`.

## Hoja de ruta

- **Encuentros salvajes / combate** — diseñado: resolución por ATK/DEF/VEL con animaciones de Ataque/Daño de PMD, rango de entrenador como final del juego. Estilo aún por elegir (automático / timing / por turnos).
- **Test de estabilidad** 24–48 h (instrumentación lista: comando `HEALTH`/latido).

*(Hecho: carcasa impresa en 3D [publicada en MakerWorld](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi); repo público con instalador para navegador + paquete de sprites en un clic.)*

## Forks de la comunidad

- **[TamaPoke — Expanded](https://github.com/ShadowEnemyx/TamaPoke/tree/tamapoke-expanded-update)** por **ShadowEnemy** — un fork importante: sistema de **combate por tipos**, los **151 + shinies** con **Pokédex / caja de colección** y objetivos diarios, **6 idiomas**, **sonido ES8311**, elección de inicial e instalador web. Vale la pena echarle un ojo. 🎮
- **[TamaPoke](https://github.com/DylanPDao/TamaPoke)** por **DylanPDao** — otro fork sustancial: **combates en gimnasios** y **combates por LAN** entre dos dispositivos, **movimientos**, sistema de **equipo + caja**, sistema de **EV/IV** y cobertura hasta la **Gen 3 (386)**. Mantiene el flujo de sprites PMD. 🏆

## Créditos

Todos los sprites: [PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab) (comunidad, CC BY-NC). Estadísticas base: [PokéAPI](https://pokeapi.co). Pokémon es una marca de Nintendo / Game Freak / The Pokémon Company. Proyecto de uso personal sin fines comerciales. Lista completa en [`CREDITS.md`](CREDITS.md). Historial de versiones en [`CHANGELOG.md`](CHANGELOG.md).

## Licencia

- **Código fuente** (firmware + herramientas): **[MIT](LICENSE)**.
- **Sprites y nombres**: © Nintendo / Game Freak / The Pokémon Company; pixel art de [PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab) (CC BY-NC 4.0). **Solo uso no comercial.**
- **Carcasa impresa en 3D**: remix de *"Pokeball"* por **yoyothechicken** ([MakerWorld #839922](https://makerworld.com/es/models/839922-pokeball)), bajo licencia **CC BY-NC-SA**, y compartida aquí en los mismos términos.

Este es un proyecto de fans no oficial, no afiliado ni respaldado por Nintendo.
