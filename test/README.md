# Tests

Batería de tests que corre **en el PC, sin placa**: compila la lógica pura del
firmware (`pet.cpp`, `i18n.cpp`, `dex.h`) contra unos shims de Arduino y la
ejecuta con un reloj y un `random()` falsos, así que los resultados son
deterministas y tardan menos de un segundo.

```bash
./test/run_tests.sh           # todo
./test/run_tests.sh --asan    # además con AddressSanitizer + UBSan
```

O por partes:

```bash
cd test && make               # solo la lógica del firmware
cd test && make FILTER=evolve # solo los tests con "evolve" en el nombre
cd test && make asan          # con sanitizers
python3 test/test_tools.py    # solo las herramientas de tools/
```

## Qué se cubre

| Fichero | Qué comprueba |
|---|---|
| `test_pet.cpp` | huevo y elección de inicial, comida, tick de estadísticas, sueño, descuidos, niveles, evolución (incluida la rama de Eevee), stats de combate, entrenamiento y minijuego, racha y vínculo, medallas, ceremonias, progresión offline y guardado en NVS |
| `test_dex.cpp` | integridad de la Pokédex: nombres ASCII, cadenas evolutivas sin bucles, toda especie alcanzable, ninguna línea imposible de completar, límites de los buffers de la UI |
| `test_i18n.cpp` | los **8 idiomas** completos, con los mismos `%u`/`%s` que le pasa el sketch en cada llamada |
| `test_tools.py` | los scripts de `tools/` compilan, los datos de la Pokédex son coherentes y `dex.h` sigue coincidiendo con lo que genera `tools/gen_dex.py` |

## Cómo está montado

- `shim/Arduino.h` + `shim/Preferences.h` + `shim/shim.cpp`: lo mínimo de Arduino
  que usan `pet.cpp` e `i18n.cpp`. El reloj (`millis()`) y `random()` son falsos
  y controlables; `Preferences` es un mapa en memoria que imita la NVS, así que
  se puede simular un reinicio creando otro `Pet` y llamando a `begin()`.
- `framework.h` + `main.cpp`: micro framework propio, sin dependencias.
- Los tests marcados con `TEST_KNOWN_ISSUE` describen el comportamiento
  **correcto** de algo que hoy falla: no rompen la batería, salen listados
  aparte, y avisan si algún día empiezan a pasar (para poder quitar la marca).

## Qué NO se cubre

Todo lo que necesita la placa: pantalla y táctil (`TamaPoke.ino`), audio
(`audio.cpp`), tarjeta SD (`sdmon.cpp`) y RTC/batería (`rtcbat.cpp`). El
`.ino` tampoco se compila aquí: hace falta `arduino-cli` con el core de ESP32.
