# Historial de cambios

Todos los cambios notables en el firmware. Las versiones coinciden con el número mostrado
en la parte inferior de la pantalla de ajustes/reloj (deslizar hacia abajo) y en `web/manifest.json`.

Actualizar desde el [instalador web](https://socquique.github.io/TamaPoke/web/)
**sin** marcar "Erase device" conserva tu Pokémon.

## [1.17] - 2026-09-18

### Añadido

- **Coreano, el octavo idioma.** Interfaz completa (85 cadenas), las tres longitudes de nombre de medalla y los nombres oficiales de las 151 especies, sobre la misma estructura `STRINGS[LANG][StrId]` que el resto de idiomas. La centralización de la medida de texto de la v1.15 se encarga de casi todo; los cuatro puntos que seguían basados en bytes han sido corregidos (ver sección de Arreglos).
- `u8g2_font_unifont_t_korean2` para Hangul, seleccionada por idioma en `applyLangFont()` en lugar de un único `CJK_FONT`. El coreano añadido usa 296 sílabas distintas, todas presentes en KS X 1001 (el conjunto de 2350 sílabas que coincide con el tamaño de `korean2`; `korean1` solo tiene 478).

### Arreglado

- **El diálogo de liberación podía cortar un nombre coreano a la mitad.** `renderRelease()` usaba `char q[28]`. `"%s 놓아줄까요?"` ocupa 17 bytes antes del nombre, por lo que nombres de cuatro sílabas o más (34 de los 151) no cabían y `snprintf` los truncaba en medio de una secuencia de 3 bytes, dejando un resto que la fuente no puede decodificar. Ahora es de 48 bytes.
- **Tres lugares medían y posicionaban el texto con `strlen()`**, que cuenta bytes: la cabecera de la ficha, el nombre de la especie bajo el apodo y la cabecera del detalle de la galería. En una fila UTF-8, el centrado y el auto-ajuste de tamaño fallaban al basarse en bytes en lugar de ancho real. Ahora los tres pasan por `textW()` / `centerX()`.
- **Tres textos en japonés se cortaban**, presentes desde la v1.15. Sus buffers se dimensionaron cuando cada carácter ocupaba un byte. El récord del minijuego solo dejaba espacio para un dígito. El banner de hito de racha no cabía en absoluto y se cortaba a la mitad de un carácter. La línea de racha en la ficha perdía dígitos a partir de los 100 días.
- **Ahora se comprueban todos los textos con formato contra su buffer en los ocho idiomas.** Un nuevo test en `test/test_tools.py` busca cada `snprintf` en el código, lee el tamaño del buffer y calcula el peor caso por idioma según los tipos de argumentos. Esto detectó los fallos anteriores y evitará regresiones.
- **`tools/test_i18n_formats.py` solo leía los primeros bloques de idiomas**, por lo que los nuevos se omitían sin aviso. Ahora lista los ocho.
- **`test/test_i18n.cpp` solo tenía seis entradas en `LANG_NAME`**, por lo que las nuevas eran nulas y provocaban fallos en los tests al pasar un puntero nulo a `%s`.

## [1.16] - 2026-09-09

### Arreglado

- **El japonés perdía silenciosamente todos los `！` y `？`.** 21 signos de exclamación y 4 de interrogación —un cuarto de las cadenas japonesas— no se mostraban porque `u8g2_font_unifont_t_japanese1` no incluye puntuación CJK (`！？。、「」`). Cambiado a `japanese3`, que sí los tiene. Cuesta 102 KB extra de flash pero no afecta a la RAM.
- **Nidoran♀ y Nidoran♂ se mostraban ambos como ニドラン en japonés**, siendo indistinguibles. Ningún subconjunto de Unifont lleva ♀ o ♂. `gen_names.py` ahora sustituye F y M también en la ruta UTF-8, por lo que ahora se leen como ニドランF y ニドランM.

## [1.15] - 2026-09-08

### Añadido

- **Japonés.** Interfaz, medallas y los 151 nombres en katakana (フシギダネ, ピカチュウ). Seleccionable desde ajustes.

### Cambiado

- La medida de texto y el posicionamiento del cursor están centralizados (`textW()`, `centerX()`, etc.), reemplazando ~180 puntos que asumían un byte por carácter. Esto permite soporte CJK sin ramas específicas por idioma en cada punto de dibujo.
- Las posiciones de las barras en la ficha se derivan ahora del ancho de la etiqueta más larga, manteniéndose alineadas en cualquier idioma.
- Nueva dependencia: **U8g2**, solo para los datos de las fuentes CJK.

## [1.11] - 2026-09-07

### Arreglado

- **El bucle principal se congelaba 1 segundo al tocar la pantalla.** `touch.getPoint()` se bloqueaba si el CST9217 estaba dormido debido al timeout por defecto del driver I2C de ESP32. Se ha corregido con una transacción previa de solo dirección.

## [1.10] - 2026-09-07

### Arreglado

- Los sprites TPK2 con duraciones de frame cero colgaban el bucle de renderizado. Ahora las duraciones se validan al cargar.
- `thumbs.bin` se mapeaba en PSRAM sin comprobar límites de tamaño, lo que podía causar lecturas fuera de memoria con archivos truncados.
- El manejador serie de `PUT` ignoraba fallos de escritura, informando de un éxito falso si la SD estaba llena. También se han restringido las escrituras a la carpeta `/mons/` por seguridad.
- Los toques justo fuera de la cuadrícula de la Pokédex o el teclado se trataban erróneamente como la celda 0 por un fallo en la división de enteros.

## [1.9] - 2026-09-07

### Arreglado

- Los temporizadores del juego fallaban tras ~49.7 días de actividad debido al desbordamiento de `millis()`.
- La transferencia de sprites ahora recuerda los archivos confirmados, evitando retransmitir los ~40 MB tras un corte de conexión.
- La física del minijuego dependía de los frames por segundo, por lo que la bola caía más lento con Pokémon grandes (más carga de renderizado) que con pequeños. Ahora depende del tiempo real.

### Seguridad

- `esp-web-tools` ahora se carga con una versión fija (10.4.0) y verificación de integridad (SRI hash).

## [1.8] - 2026-09-07

### Arreglado

- **El contador de nivel volvía a 0 tras ~10.6 días de juego** (desbordamiento de `uint8_t` a los 256 niveles/horas). Ahora es `uint16_t` con límite en 999.
- Se han corregido otros desbordamientos relacionados en el cálculo de estadísticas y la comprobación de evolución que podían causar comportamientos erráticos tras mucho tiempo de juego o muchos descuidos.

## [1.7] - 2026-09-07

### Arreglado

- **La interfaz no tenía acentos.** Se pensaba que la fuente no los soportaba, pero `glcdfont.h` es una tabla CP437 completa. El español ahora tiene tildes y eñes, el alemán diéresis y el francés sus acentos.
- Correcciones gramaticales en los textos en alemán.

## [1.6] - 2026-09-07

### Arreglado

- **Los efectos de sonido cortos no se oían.** El amplificador se encendía solo 8 ms antes del efecto, tiempo insuficiente para estabilizarse. Ahora el amplificador se mantiene encendido mientras el sonido está activo y la mascota despierta.

## [1.5] - 2026-08-19

### Añadido

- Nombres de Pokémon localizados en francés y alemán (Bulbizarre, Bisasam...). Español, italiano y portugués usan oficialmente los nombres en inglés en la primera generación.

## [1.4] - 2026-08-07

### Arreglado

- **El vínculo estaba matemáticamente atascado.** Los descuidos penalizaban más de lo que se podía ganar en un día. Se han ajustado los límites y penalizaciones para que la recuperación sea posible.

## [1.3] - 2026-08-07

### Arreglado

- **El huevo eclosionaba solo durante la selección del inicial.** El tiempo seguía corriendo en la pantalla de selección; ahora se detiene hasta que el jugador elige su primer Pokémon.

## [1.2] - 2026-06

### Añadido

- Versión del firmware visible en la parte inferior de la pantalla de ajustes/reloj.
