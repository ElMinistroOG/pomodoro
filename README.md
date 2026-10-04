# Pomodoro

Timer método pomodoro con perilla. Web app (PWA) para escritorio y celular (en celular, solo horizontal).

## Publicarla gratis con GitHub Pages

1. Crea una cuenta en github.com si no tienes.
2. Arriba a la derecha: **+ → New repository**. Nombre: `pomodoro`. Márcalo como **Public** y pulsa **Create repository**.
3. En la página del repositorio pulsa **uploading an existing file**.
4. Arrastra **el contenido** de esta carpeta (index.html, manifest.json, sw.js y la carpeta icons), no la carpeta entera. Pulsa **Commit changes**.
5. Ve a **Settings → Pages**. En *Branch* elige `main` y `/ (root)`, y pulsa **Save**.
6. Espera uno o dos minutos. Tu app quedará en `https://TU-USUARIO.github.io/pomodoro/`.

## Instalarla

- **iPhone / iPad:** abre el enlace en Safari → botón compartir → **Agregar a pantalla de inicio**.
- **Mac:** en Safari → menú Archivo → **Agregar al Dock**. En Chrome, el icono de instalar en la barra de direcciones.
- **Android:** Chrome → menú ⋮ → **Instalar app**.

Una vez abierta la primera vez, funciona sin conexión.

## Actualizarla

Sube los archivos nuevos al mismo repositorio (reemplazando los anteriores) y, en `sw.js`, cambia `pomodoro-v1` por `pomodoro-v2` (luego v3, etc.). Así los dispositivos instalados descargan la versión nueva. Si no se actualiza, cierra la app por completo y vuelve a abrirla.

## Archivos

- `index.html`: la app completa.
- `manifest.json`: nombre, icono y orientación horizontal al instalarla.
- `sw.js`: permite usarla sin conexión.
- `icons/`: iconos (el de 1024 px sirve para la App Store o cualquier otro uso).
