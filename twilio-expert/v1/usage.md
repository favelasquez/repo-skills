---
metadata:
  author: https://github.com/favelasquez
name: twilio-expert
description: >
  Skill experta en Twilio cubriendo Voice (llamadas, TwiML, Media Streams),
  SMS, WhatsApp, AMD (Answering Machine Detection), Webhooks y Virtual Agents.
  Activar siempre que el usuario mencione Twilio, TwiML, Media Streams, llamadas
  salientes, llamadas entrantes, AMD, buzÃ³n de voz, IVR, Twilio Verify, Twilio
  Lookup, StatusCallback, webhook de Twilio, mulaw, stream de audio, Twilio REST
  API, twilio-python, twilio-node, o cualquier concepto del ecosistema Twilio.
  TambiÃ©n activar cuando el usuario tenga problemas con llamadas que se caen,
  AMD que no detecta buzÃ³n, TwiML que no funciona, Media Streams que se
  desconectan, o webhooks que no llegan. Esta skill detecta configuraciones
  incorrectas, antipatrones y bugs en integraciones Twilio con ejemplos
  corregidos listos para producciÃ³n.
---
metadata:
  author: https://github.com/favelasquez

# Twilio â€” Skill Experta Completa

## Productos cubiertos

| Producto | DescripciÃ³n | Referencia |
|---|---|---|
| **Voice + TwiML** | Control de flujo de llamadas | `references/twiml-voice.md` |
| **Media Streams** | Audio bidireccional en tiempo real | `references/media-streams.md` |
| **AMD** | DetecciÃ³n de buzÃ³n de voz | `references/amd.md` |
| **SMS / WhatsApp** | MensajerÃ­a | `references/sms-whatsapp.md` |
| **Webhooks y callbacks** | Eventos, errores, estado | `references/webhooks.md` |

---
metadata:
  author: https://github.com/favelasquez

## Paso 1 â€” DiagnÃ³stico rÃ¡pido

```
Â¿QuÃ© tipo de llamada?
  Entrante  â†’ Twilio llama a tu webhook con el CallSid â†’ tu servidor responde TwiML
  Saliente  â†’ Tu servidor llama a Twilio REST API â†’ Twilio llama al nÃºmero destino

Â¿CuÃ¡l es el problema?
  Llamada se cae sola          â†’ TwiML sin <Pause> suficiente o WebSocket sin ping
  AMD no detecta buzÃ³n         â†’ machine_detection_timeout corto o DetectMessageEnd mal configurado
  Audio corrupto               â†’ encoding/sample_rate incorrecto en Media Streams
  Webhook no llega             â†’ URL no pÃºblica, falta HTTPS, o ngrok expirÃ³
  Media Stream se desconecta   â†’ Sin keep-alive o servidor WebSocket sin ping_interval
  TwiML no hace nada           â†’ Content-Type debe ser application/xml, no text/html
  "Cita confirmada" pero no cuelga â†’ Falta <Hangup/> o no se llama calls(sid).update()

Â¿QuÃ© SDK usas?
  from twilio.rest import Client     â†’ Python (twilio>=8.0)
  const twilio = require('twilio')   â†’ Node.js
  TwiML via REST API                 â†’ cualquier lenguaje con HTTP
```

---
metadata:
  author: https://github.com/favelasquez

## Flujo de llamada saliente con Virtual Agent â€” mapa completo

```
Tu servidor
    â”‚
    â”œâ”€ 1. calls.create(to, from, url, machine_detection) â”€â”€â†’ Twilio REST API
    â”‚                                                              â”‚
    â”‚  â†â”€â”€ CallSid devuelto inmediatamente                        â”‚
    â”‚                                                          Twilio marca
    â”‚                                                              â”‚
    â”‚  â†â”€â”€ POST /twiml/outbound-hold (TwiML inicial) â†â”€â”€â”€â”€â”€â”€ TelÃ©fono contesta
    â”‚      Respuesta: <Pause length="60"/>                         â”‚
    â”‚                                                          AMD trabajando...
    â”‚  â†â”€â”€ POST /amd-result (AnsweredBy=human/machine_end_beep)    â”‚
    â”‚      â†’ Si humano: update(twiml con <Stream>)                 â”‚
    â”‚      â†’ Si buzÃ³n:  update(twiml con <Say>+<Hangup/>)          â”‚
    â”‚                                                              â”‚
    â”‚  â†â”€â”€ WebSocket /ws/agent â†â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ Media Stream abierto
    â”‚      Audio Î¼-law 8kHz bidireccional                          â”‚
    â”‚                                                              â”‚
    â”‚  â†â”€â”€ POST /call-status (completed/failed/busy)          Llamada termina
```

---
metadata:
  author: https://github.com/favelasquez

## Checklist de auditorÃ­a â€” ejecutar en cada revisiÃ³n

### ðŸ”´ Llamadas y TwiML

- [ ] Â¿El webhook de Twilio responde con `Content-Type: application/xml`?
- [ ] Â¿El TwiML de llamadas salientes tiene `<Pause length="60"/>` para mantener la llamada?
- [ ] Â¿Se usa `<Hangup/>` para colgar o `calls(sid).update()` desde el servidor?
- [ ] Â¿Las URLs de webhook son HTTPS y pÃºblicas? (no localhost)
- [ ] Â¿El StatusCallback estÃ¡ configurado para recibir eventos de la llamada?

### ðŸ”´ AMD (Answering Machine Detection)

- [ ] Â¿Se usa `machine_detection="DetectMessageEnd"` (no solo `"Enable"`)?
- [ ] Â¿`machine_detection_timeout` es al menos 45 segundos?
- [ ] Â¿`async_amd=True` para no bloquear la llamada mientras detecta?
- [ ] Â¿El callback AMD maneja `human`, `machine_end_beep`, `machine_end_silence`, `machine_end_other`, `unknown` y `fax`?
- [ ] Â¿`unknown` tiene fallback (tratar como humano)?

### ðŸ”´ Media Streams

- [ ] Â¿El WebSocket del servidor tiene `ping_interval` configurado?
- [ ] Â¿El audio enviado de vuelta a Twilio estÃ¡ en formato Î¼-law 8kHz?
- [ ] Â¿Se envÃ­a `{"event":"clear","streamSid":sid}` al interrumpir el TTS (barge-in)?
- [ ] Â¿Se manejan los eventos `connected`, `start`, `media`, `stop` de Twilio?
- [ ] Â¿El audio de Twilio se decodifica de Base64 antes de procesarlo?
- [ ] Â¿El `streamSid` se captura del evento `start` para usarlo en respuestas?

### ðŸŸ  Webhooks y seguridad

- [ ] Â¿Se valida la firma de Twilio en los webhooks? (`X-Twilio-Signature`)
- [ ] Â¿Las credenciales estÃ¡n en variables de entorno, no en cÃ³digo?
- [ ] Â¿Los webhooks responden en menos de 15 segundos? (timeout de Twilio)
- [ ] Â¿Hay retry logic si el webhook falla?

---
metadata:
  author: https://github.com/favelasquez

## Bugs mÃ¡s comunes â€” referencia rÃ¡pida

### 1. Content-Type incorrecto â€” TwiML ignorado silenciosamente

```python
# âŒ Twilio recibe HTML/JSON en lugar de XML â†’ cuelga la llamada
@app.post("/webhook")
async def webhook():
    return {"twiml": "<Response><Say>Hola</Say></Response>"}  # JSON â€” incorrecto

# âœ… Siempre application/xml
from fastapi.responses import Response

@app.post("/webhook")
async def webhook():
    twiml = """<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <Say voice="Polly.Lupe" language="es-US">Hola, Â¿cÃ³mo puedo ayudarte?</Say>
    <Pause length="60"/>
</Response>"""
    return Response(content=twiml, media_type="application/xml")
```

### 2. Llamada se cae inmediatamente â€” TwiML sin pausa

```xml
<!-- âŒ Twilio ejecuta el Say y cuelga â€” sin tiempo para conversaciÃ³n -->
<Response>
    <Start><Stream url="wss://servidor.com/ws"/></Start>
    <Say>Hola, estoy aquÃ­ para ayudarte</Say>
</Response>
<!-- Cuando termina el <Say>, Twilio cuelga â†’ WebSocket se cierra -->

<!-- âœ… <Pause> para mantener la llamada abierta -->
<Response>
    <Start><Stream url="wss://servidor.com/ws"/></Start>
    <Pause length="120"/>
</Response>
<!-- 120s de pausa â†’ la llamada dura hasta que el servidor la cuelgue -->
```

### 3. Colgar desde el servidor â€” forma correcta

```python
# âŒ Solo cerrar el WebSocket â€” Twilio puede tardar 5s en detectarlo
await websocket.close()

# âŒ Responder TwiML con Hangup â€” solo funciona en respuesta directa al webhook
# No funciona para colgar desde el servidor en cualquier momento

# âœ… Llamar a la REST API â€” instantÃ¡neo
from twilio.rest import Client

twilio = Client(
    os.environ["TWILIO_ACCOUNT_SID"],
    os.environ["TWILIO_AUTH_TOKEN"]
)

def hang_up(call_sid: str):
    twilio.calls(call_sid).update(
        twiml='<?xml version="1.0"?><Response><Hangup/></Response>'
    )
    print(f"Llamada {call_sid} colgada")
```

### 4. Media Stream â€” audio enviado antes de que abra

```python
# âŒ El evento 'start' aÃºn no llegÃ³ â€” stream_sid desconocido
async def send_audio_to_twilio(ws, audio_bytes: bytes):
    await ws.send(json.dumps({
        "event": "media",
        "streamSid": None,  # â† None porque 'start' no llegÃ³ aÃºn
        "media": {"payload": base64.b64encode(audio_bytes).decode()}
    }))

# âœ… Capturar stream_sid en el evento 'start' antes de enviar audio
stream_sid = None

async for message in twilio_ws:
    packet = json.loads(message)
    event = packet.get("event")

    if event == "start":
        stream_sid = packet["start"]["streamSid"]
        print(f"Stream abierto: {stream_sid}")

    elif event == "media" and stream_sid:
        # Ahora sÃ­ tenemos el streamSid â€” procesar y responder
        audio = base64.b64decode(packet["media"]["payload"])
        # ... procesar audio ...

    elif event == "stop":
        stream_sid = None
        break
```

### 5. Barge-in â€” vaciar buffer de Twilio

```python
# âŒ Cancelar el TTS pero el audio ya enviado sigue sonando en Twilio
tts_task.cancel()  # cancela la generaciÃ³n, pero Twilio ya tiene audio en buffer

# âœ… Enviar 'clear' + cancelar el task
async def interrupt_agent(twilio_ws, stream_sid: str, tts_task: asyncio.Task):
    # 1. Vaciar el buffer de audio de Twilio
    await twilio_ws.send(json.dumps({
        "event": "clear",
        "streamSid": stream_sid
    }))
    # 2. Cancelar la generaciÃ³n de TTS
    if tts_task and not tts_task.done():
        tts_task.cancel()
    print("Buffer vaciado + TTS cancelado")
```

### 6. Validar firma de Twilio â€” webhooks falsos

```python
# âŒ Sin validaciÃ³n â€” cualquiera puede enviar requests a tu webhook
@app.post("/webhook")
async def webhook(request: Request):
    form = await request.form()
    # procesar sin verificar que viene de Twilio

# âœ… Validar X-Twilio-Signature
from twilio.request_validator import RequestValidator

validator = RequestValidator(os.environ["TWILIO_AUTH_TOKEN"])

async def verify_twilio_signature(request: Request) -> bool:
    signature = request.headers.get("X-Twilio-Signature", "")
    url = str(request.url)
    form = await request.form()
    params = dict(form)
    return validator.validate(url, params, signature)

@app.post("/webhook")
async def webhook(request: Request):
    if not await verify_twilio_signature(request):
        raise HTTPException(status_code=403, detail="Firma invÃ¡lida")
    # procesar con seguridad
```

---
metadata:
  author: https://github.com/favelasquez

## Variables de entorno requeridas

```bash
# .env
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_PHONE_NUMBER=+15551234567
SERVER_HOST=tu-servidor.com   # para construir URLs de webhook
```

---
metadata:
  author: https://github.com/favelasquez

Leer archivos `references/` para guÃ­as completas por Ã¡rea.

