# 🎅 Viejito Pascuero IA · App móvil familiar 2.0

**Una aplicación web instalable (PWA) para Android/iPhone**, con personaje ilustrado de cuerpo completo y audio bidireccional por OpenAI Realtime WebRTC. No es un APK nativo ni genera video realista con labios sincronizados: Santa se balancea al hablar. Compatible con navegador moderno y HTTPS.

## Qué hay construido

- Santa de cuerpo completo en taller navideño, con movimiento suave al responder.
- Una página responsive para teléfono/tablet/PC, e iconos y manifiesto PWA para añadir a pantalla de inicio.
- Conversación real por voz con **OpenAI Realtime API** al configurar tu clave y crédito API; se usa WebRTC, el servidor solo crea credenciales temporales.
- **Modo demostración**: voces y respuestas guionizadas, sin API; reconocimiento de voz solo si el navegador lo admite.
- Nombre/edad/intereses editables, 4 juegos rápidos, controles de micrófono y colgar.
- Consentimiento del adulto, contraseña familiar de acceso, limitación de 5 solicitudes de sesión/hora, fin automático en la interfaz a los 6 minutos.
- No usa cámara, no guarda audios/transcripciones en archivos o bases de datos de este proyecto. *La voz se procesa por servicios externos según sus políticas.*

## Requisitos

- **Node.js 22.6+** en el computador o alojamiento web.
- Para activar la IA: una **clave OpenAI API**, con acceso a `gpt-realtime-2.1` y facturación/créditos disponibles. La API se cobra independientemente de ChatGPT Plus/Pro.
- Para usar micrófono desde un teléfono: **HTTPS** en el sitio alojado, o `localhost` en el mismo dispositivo. Una web servida por IP local sin HTTPS no suele poder pedir micrófono.
- Un adulto presente durante la interacción.

## Opción A: probar en un computador (sin clave)

1. Instala Node.js 22.6+ desde https://nodejs.org.
2. Descomprime el ZIP, abre una consola en esta carpeta.
3. Ejecuta `npm start`.
4. Abre `http://localhost:8765`.
5. Marca la casilla de adulto y toca **Llamar a Santa**. Funciona en modo demostración con juegos preparados. Con Chrome o Edge puede utilizar reconocimiento de voz; si no, usa los botones de juego.

Para modo IA local, crea `.env` a partir de `.env.example` y asigna `OPENAI_API_KEY=tu_clave`. Reinicia con `npm start`. **No compartas este archivo.**

## Opción B: usarlo como app en Android o iPhone con HTTPS

La forma más simple es alojar el servidor en un servicio compatible con **Node.js**, por ejemplo Render o Railway. El proyecto trae `render.yaml` como ejemplo (comprueba los planes y precios vigentes).

1. Sube esta carpeta como repositorio **privado** a GitHub. Asegúrate de NO subir `.env`; el `.gitignore` lo excluye.
2. En Render crea un **Web Service** desde ese repositorio, o utiliza el archivo `render.yaml`.
3. Comando de inicio `npm start`. Versión de Node 22+. Configura variables de entorno:
   - `HOST=0.0.0.0`
   - `FAMILY_PASSWORD=una contraseña única de 10 o más caracteres`
   - `OPENAI_API_KEY=tu clave API` (si quieres IA real)
   - `OPENAI_REALTIME_MODEL=gpt-realtime-2.1`
   - `OPENAI_REALTIME_VOICE=cedar`
   - `PORT` normalmente lo proporciona el proveedor; no lo fuerces si el proveedor lo configura.
4. Abre la **URL HTTPS** creada por el proveedor en Chrome de Android o Safari de iPhone.
5. Ingresa la contraseña familiar, permite el micrófono cuando presiones «Llamar a Santa» y prueba con un adulto.
6. Instala desde el menú del navegador: Android → ⋮ → **Instalar aplicación** o **Agregar a pantalla principal**. iPhone → Compartir → **Añadir a pantalla de inicio**.

La publicación web **no se hizo desde esta conversación**: se necesita tu cuenta de hosting y la clave API. Nunca pegues la clave dentro de HTML, JS del navegador ni en el chat.

## Arquitectura

```
Teléfono (PWA)                  Servidor privado Node.js          OpenAI
 ├─ imagen Santa                  ├─ acceso familiar                 │
 ├─ micrófono ↔ WebRTC ──────────────────────────────────────────→ Realtime
 ├─ audio de respuesta           └─ /api/session ─ token temporal ─→ API
 └─ juegos/controles
```

**El audio WebRTC va del dispositivo a OpenAI:** el servidor propio solicita tokens de corta duración usando la clave principal, sin enviarla al navegador.

## Seguridad y límites

- Solo la familia debe tener la contraseña; usa una **contraseña larga y única**, no una clave de API.
- **No publiques el servidor por HTTP**. La contraseña y el micrófono requieren HTTPS para uso seguro desde móviles.
- El servidor bloquea accesos sin contraseña y limita solicitudes a la API. Los límites son en memoria, **no sustituyen** los límites de gasto de tu cuenta OpenAI.
- El límite de 6 minutos es aplicado por el cliente. Para control estricto de costo y abuso haría falta una arquitectura con controles server-side adicionales y presupuestos API.
- Usar niños en una app de voz implica revisar la política de OpenAI y obtener consentimiento de sus responsables; no lo despliegues públicamente ni permitas que menores lo usen solos.
- El código de esta app no guarda grabaciones. La API y el reconocimiento de voz del navegador pueden procesar audio conforme a las condiciones de cada proveedor.
- No se realiza monitoreo parental automático del contenido; las instrucciones son orientativas y no garantizan que el modelo nunca se equivoque.
- La contraseña de acceso y la clave API son cosas distintas.

## Pruebas

- `npm test` prueba sanitización, acceso, limitación de páginas, login y token usando simulación de OpenAI (sin hacer llamadas facturables).
- Sin clave no es posible probar aquí una conversación de IA real de extremo a extremo.
- Para verificar en el móvil: prueba que el sitio abra por HTTPS, permite micrófono, inicia llamada, habla, interrumpe a Santa, silencia y cuelga, y comprueba el corte a los 6 minutos.

## Documentación de la API

- https://developers.openai.com/api/docs/guides/realtime
- https://developers.openai.com/api/docs/guides/voice-webrtc
- https://developers.openai.com/api/reference/resources/realtime/subresources/client_secrets/methods/create

## Archivos

- `server.js`: backend HTTP privado y credenciales temporales.
- `public/index.html`, `public/style.css`, `public/app.js`: interfaz y voz.
- `public/login.html`, `public/login.js`: acceso privado.
- `public/manifest.webmanifest`, `public/sw.js`, `public/icons/`: instalación PWA.
- `public/assets/santa.webp`: versión optimizada; `santa-original.png`: arte original.
- `.env.example`: variables a copiar a un `.env` privado.
- `render.yaml`: ejemplo de despliegue.
