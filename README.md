# Zelda: Echoes of Wisdom — port para PC (Windows)

Port nativo para Windows de *The Legend of Zelda: Echoes of Wisdom* (Switch, versión 1.0.0) mediante
**recompilación estática**: el código del juego se traduce a C y se compila como un programa de PC.
Construido sobre **SwitchRecomp by MZ Labs** (incluido en `tools/SwitchRECOMP`).

> **Este repositorio no contiene el juego.** No incluye datos, código ni recursos de Nintendo. Para jugar
> necesitas **tu propia copia original** (NSP de la versión 1.0.0) y las claves (`prod.keys`) **de tu propia
> consola**. No pidas ni compartas juegos ni claves.

## Qué añade este port

- **Launcher** propio (Win32 + GDI+): resolución, relación de aspecto (16:9, 16:10, 21:9), campo de visión,
  FPS, gráficos, controles, idioma y audio.
- **Agregar copia del juego**: el launcher extrae tu NSP con tus claves (`nsp_import`), comprueba que sea la
  1.0.0 y deja los datos listos.
- **Renderer Vulkan** con correcciones para este juego: menú de pausa, efectos compute, copias 2D volteadas.
- **60 y 120 FPS reales** (el movimiento del juego va por tiempo, la velocidad es la normal), presentación
  asíncrona y vsync preciso a 120 Hz.
- **Calidad**: 3D a 1080p nativo (la consola usa 1408x792), FXAA, nitidez AMD FidelityFX CAS, opción para
  quitar el desenfoque tilt-shift / profundidad de campo, compatible con el escalado de Windows (DPI).
- **Mandos** vía SDL3 (Switch Pro, DualSense/DualShock, Xbox) con **iconos de botones y dibujo del mando** del
  mando que estés usando (arte propio), vibración y corrección de drift.
- **Estabilidad**: arreglo de una señal perdida en las variables de condición del kernel emulado (cuelgue al
  entrar al calabozo del castillo), de un interbloqueo al entrar en la selección de partida, y más.

La lista de cambios sobre SwitchRecomp v190 está en
[`tools/SwitchRECOMP/CHANGES_ZELDA_EOW.txt`](tools/SwitchRECOMP/CHANGES_ZELDA_EOW.txt).

## Estructura

| Carpeta | Contenido |
|---|---|
| `tools/SwitchRECOMP/` | El recompilador y el runtime (AGPL-3.0) con los cambios del port. Renderer Vulkan en `runtime/snxvk/` (GPL-3.0). |
| `launcher/` | El launcher (`launcher.cpp`, `build.bat`). |
| `tools/nxextract/` | `nsp_import.cpp` (el importador del launcher, sin dependencias), `nxextract.py` (versión Python). |
| `tools/re/` | Herramientas: `make_icons.py` (iconos y dibujos de mandos), lectores de BNTX/SARC/BFLYT/BFFNT, `make_release.ps1`, scripts de prueba. |
| `game/` | `ZeldaEoW.cfgpatch` (parches de aspecto/FOV: solo direcciones y valores) y `ZeldaEoW.hints`. |
| `release/LEEME.txt` | Instrucciones para jugadores (español). |

## Compilar

Requisitos: Windows 10/11 x64, Python 3.11, [Vulkan SDK](https://vulkan.lunarg.com/) 1.4.x, tu NSP 1.0.0 y tus `prod.keys`.

1. **Herramientas de SwitchRecomp**: ejecuta `tools/SwitchRECOMP/Install.bat` (descarga GCC y lo necesario
   a `tools/SwitchRECOMP/gcc`).
2. **Dependencias del renderer** en `tools/SwitchRECOMP/runtime/snxvk/deps/`: `volk-vulkan-sdk-1.4.363.0`,
   `VulkanMemoryAllocator-3.4.0` y `Vulkan-Headers-1.4.365` (del Vulkan SDK / sus repositorios). `glslang`
   del SDK en el PATH.
3. **VP9 (vídeos)**: libvpx 1.14.0 compilado como `tools/SwitchRECOMP/runtime/vp9/win/libvpx.a`.
4. **Renderer**:
   ```bat
   cd tools\SwitchRECOMP\runtime\snxvk
   set SWR_NO_DLSS=1
   python build_lib.py
   ```
5. **Datos del juego** (tu copia): compila el launcher (paso 7) y usa "Agregar copia del juego", o
   ```bat
   nsp_import.exe "tu_juego.nsp" "prod.keys" games\ZeldaEoW\ZeldaEoW_data
   ```
6. **Recompilar el juego** (copia antes `game\ZeldaEoW.cfgpatch` y `game\ZeldaEoW.hints` a `games\ZeldaEoW\`):
   ```bat
   cd tools\SwitchRECOMP
   python switchrecomp.py ..\..\games\ZeldaEoW\ZeldaEoW_data -o ZeldaEoW --outdir ..\..\games\ZeldaEoW --cc gcc\Library\bin\x86_64-w64-mingw32-gcc.exe
   ```
   Tras cambiar solo el runtime basta con
   `python switchrecomp.py --build-runtime --target win64 --backend win32 --cc ...` y copiar
   `bin\runtime_win64_win32.exe` como `games\ZeldaEoW\ZeldaEoW.exe`.
7. **Launcher**: `launcher\build.bat` (compila también `nsp_import.exe`). Necesita `launcher\assets\bg.png`
   (fondo de 1586x992) y `launcher\assets\icon.ico`, que no se incluyen en el repositorio.

Lo generado a partir del juego (`ZeldaEoW_code.dll`, `ZeldaEoW_build/`, `controller_icons/`) queda en
`games/` y **nunca** debe subirse: el `.gitignore` lo bloquea.

## Licencias y créditos

- **SwitchRecomp by MZ Labs** — AGPL-3.0 con términos adicionales: ver
  [`tools/SwitchRECOMP/LICENSE.txt`](tools/SwitchRECOMP/LICENSE.txt) y
  [`tools/SwitchRECOMP/NOTICE.txt`](tools/SwitchRECOMP/NOTICE.txt) (la pantalla de inicio de SwitchRECOMP se
  mantiene).
- Renderer Vulkan (ScarletNX / snxvk) — GPL-3.0: `tools/SwitchRECOMP/runtime/snxvk/LICENSE` y `NOTICE`.
- El resto de este repositorio (launcher, herramientas, cambios del port) se publica bajo **AGPL-3.0**
  ([`LICENSE`](LICENSE)).
- *The Legend of Zelda* y *Echoes of Wisdom* son marcas de Nintendo. Proyecto de fans sin relación con
  Nintendo; no incluye ningún material del juego.
