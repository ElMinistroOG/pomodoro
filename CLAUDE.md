# CLAUDE.md — Pomodoro

Lee este archivo entero antes de tocar nada. Está escrito para que un chat sin contexto pueda seguir el proyecto.

## 1. Qué es
**Pomodoro**: temporizador del método pomodoro con una perilla (dial) física simulada, estilo dispositivo de hardware con pantalla LCD de 7 segmentos. Es una **PWA** (web app instalable, funciona sin conexión) para escritorio y móvil (en móvil solo en horizontal). Idioma de la interfaz: **español**. Es una app personal del usuario (aarhonale@gmail.com).

- Web publicada: https://elministroog.github.io/pomodoro/
- Repositorio: https://github.com/ElMinistroOG/pomodoro (público, rama `main`, GitHub Pages desde `/` de `main`)

## 2. Tecnologías y dependencias
- HTML + CSS + JavaScript puro, **sin build, sin npm, sin frameworks**. Todo el código de la app está en un único `index.html` (~537 líneas, ~43 KB).
- Fuente LCD **DSEG7 incrustada en base64** (woff2) dentro del CSS de `index.html` (no hay archivo de fuente aparte).
- Única dependencia externa: Google Fonts **Inter** (`<link>` en `<head>`). Si falla la red, hay fallback a fuentes del sistema (`--ui`).
- APIs del navegador usadas: Service Worker, Web Audio (sonidos generados por código, sin archivos de audio), Screen Wake Lock, Vibration, Notifications (ver pendientes), localStorage, Pointer Events.

## 3. Estructura de archivos
| Archivo | Función |
|---|---|
| `index.html` | La app completa: estilos (temas por variables CSS), marcado y script en IIFE. |
| `manifest.json` | Nombre, iconos, `display: standalone`, `orientation: landscape`, colores. |
| `sw.js` | Service worker: precachea los archivos y sirve cache-first con actualización en segundo plano. Constante `VERSION` (`pomodoro-v1`). |
| `icons/` | `icon-96/180/192/512/1024.png`. El de 1024 es para App Store u otros usos; no está en el precache. |
| `README.md` | Guía para el usuario: publicar en GitHub Pages, instalar, actualizar. |
| `.gitignore` | `.DS_Store`, `*.log`, `.env`, `.env.*`. |

### Dentro de `index.html` (orden)
1. `<head>`: metas PWA/iOS, fuentes, CSS (tokens de color por tema, neumorfismo con sombras `--raise*`, `--sink`, `--press`).
2. `<body>`: cabecera con menú (Tema, Sonido, Ciclo), `.device` con LCD (`#lcd`, `#time`, rondas, 4 LEDs de ciclo), perilla `#dial` con anillo SVG de 300° (`#arc`, `#glow`), presets (`#presets`) y transporte (reset / play / skip), aviso `.rotate` para vertical en móvil, `#toast`.
3. `<script>`: SW, `store` (localStorage con prefijo `pomodoro-`), temas, estado `st`, audio (`bell`, `noiseClick`), wake lock, render (`paint`, `ui`, `loop`), lógica de fases (`start`, `pause`, `nextPhase`, `finish`), perilla (`setMinutes`, eventos pointer/wheel/teclado), atajos y arranque.

## 4. Funcionamiento
- **Fases**: foco → pausa corta → foco … tras **4 focos completados** viene pausa larga y el contador se reinicia. Al terminar una fase **no arranca sola**: hay que pulsar iniciar (decisión de diseño; el aviso lo dice).
- **Presets** (foco/pausa corta/pausa larga en min): Clásico 25/5/15, Deep work 90/20/30, Corto 15/3/10.
- **Perilla**: ajusta los minutos de la fase actual, 1–90 min (300° de recorrido). Arrastre con puntero, rueda del ratón, teclado (flechas ±1, PgUp/PgDn ±5, Home/End). Cambiar minutos se guarda en `cfg` de esa fase.
- **Atajos**: Espacio iniciar/pausar, `R` reiniciar fase, `N` saltar a la siguiente fase (no cuenta como foco completado), `Esc` cierra el menú.
- **Temas**: `light`, `mist`, `dark` (Ultra dark), `space` (Sideral Space). Por defecto según `prefers-color-scheme`. Se guarda en localStorage.
- **Persistencia**: estado completo (`cfg`, fase, `done`, `running`, `endAt`, `remaining`), tema y sonido en localStorage.

## 5. Ejecutar en local
No hay build. Para probar con Service Worker hace falta servirla por HTTP (el SW solo se registra si `location.protocol` empieza por `http`):
```bash
cd "<carpeta del proyecto>" && python3 -m http.server 8000
# abrir http://localhost:8000
```
También se puede abrir `index.html` directamente (funciona, pero sin modo offline).

## 6. Publicar / actualizar
- Hosting: **GitHub Pages**, rama `main`, carpeta raíz. Cuenta de GitHub: **`ElMinistroOG`** (la sesión de `gh` estaba logueada con esa cuenta, no con `aarhonale`; el usuario fue avisado y no pidió moverlo).
- Publicar cambios:
```bash
git add -A && git commit -m "mensaje" && git push
```
Pages se redespliega solo en 1–2 min.
- **IMPORTANTE al cambiar archivos de la app**: sube `VERSION` en `sw.js` (`pomodoro-v1` → `v2`, …). Si no, los dispositivos instalados siguen sirviendo la versión en caché. Si aun así no se actualiza: cerrar la app por completo y reabrir.
- Si añades archivos nuevos que deban funcionar offline, agrégalos al array `FILES` de `sw.js`.

## 7. Decisiones tomadas y por qué
- **Un solo `index.html`, sin dependencias ni build**: máxima simplicidad para publicar subiendo archivos a GitHub Pages desde el navegador (así lo describe el README).
- **Fuente LCD en base64**: evita una petición externa y funciona offline.
- **Sonidos con Web Audio**: sin archivos de audio que cachear.
- **Timestamp `endAt` en vez de contar ticks**: el tiempo restante se calcula con `Date.now()`, así sobrevive a pestañas en segundo plano y a recargas; al volver a la pestaña (`visibilitychange`) se recalcula o se termina la fase.
- **Fases no arrancan solas**: evita que el usuario empiece un foco sin querer; el toast indica pulsar iniciar.
- **Solo horizontal en móvil**: el diseño del dispositivo no cabe en vertical; `manifest` fuerza landscape y hay un aviso "Gira el teléfono" (`.rotate`, media query `orientation: portrait` y `max-width: 600px`).
- **Wake Lock** mientras corre, para que la pantalla no se apague.
- Respeta `prefers-reduced-motion`.

## 8. Limitaciones conocidas y cosas a no tocar
- **Pantalla bloqueada = no suena a tiempo**: los navegadores suspenden JS y audio; es una limitación de la plataforma, no un bug. Está avisado en el menú ("Ciclo"). No intentar arreglarlo con hacks de audio silencioso sin discutirlo.
- **No renombrar** el prefijo `pomodoro-` de localStorage ni la forma del objeto `state`: se perdería el estado guardado de los usuarios/dispositivos.
- **No tocar** la geometría del arco (`M136 310.85 A128 128 0 1 1 264 310.85`, `pathLength="300"`) sin recalcular el mapeo de 300° ↔ 90 min en `setMinutes`/`paint`.
- **No poner `purpose: "maskable"` con iconos que no tengan margen seguro**: hoy `icon-512.png` se usa como `any` y `maskable` a la vez (puede recortarse en Android; ver pendientes).
- `index.html` pesa mucho por la fuente base64: al editar, usa búsquedas/líneas concretas, no vuelques el archivo entero.
- El historial git solo tiene un commit (`Initial commit: Pomodoro app`); no hay ramas ni historial de versiones intermedias. La versión actual de `main` (= `origin/main`, árbol limpio) es la definitiva.

## 9. Estado actual
**Funciona** (según código revisado; no se ha hecho una batería de pruebas en dispositivos reales en esta sesión de cierre):
- Temporizador completo con ciclo de 4 focos, presets, perilla, atajos, 4 temas, sonido on/off, persistencia, PWA offline, publicado en GitHub Pages.

**Sin verificar / posibles fallos**:
- Comportamiento de alarma con la app en segundo plano en iOS.
- Notificaciones: el código las muestra solo si `Notification.permission === 'granted'`, pero **nunca se solicita el permiso** (`requestPermission` no aparece), así que en la práctica no salen.

## 10. Pendientes e ideas
- Pedir permiso de notificaciones (p. ej. al primer inicio o con un botón en el menú).
- Icono maskable propio con margen seguro; añadir `icon-1024` solo si se necesita.
- Añadir `<meta name="description">` y capturas al README.
- Estadísticas simples (focos completados por día) en localStorage.
- Opción de arranque automático de la siguiente fase (opcional, desactivada por defecto).
- Ajustar duración de pausa corta/larga desde la perilla con selector de fase explícito (hoy la perilla cambia la fase en curso).
- Considerar mover el repo a la cuenta `aarhonale` si el usuario lo prefiere.
