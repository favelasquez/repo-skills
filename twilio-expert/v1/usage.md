---
metadata:
  version: "1.0"
  author: https://github.com/favelasquez
name: twilio-expert
description: >
  Skill experta en Twilio cubriendo Voice (llamadas, TwiML, Media Streams),
  SMS, WhatsApp, AMD (Answering Machine Detection), Webhooks y Virtual Agents.
  Activar siempre que el usuario mencione Twilio, TwiML, Media Streams, llamadas
  salientes, llamadas entrantes, AMD, buzón de voz, IVR, Twilio Verify, Twilio
  Lookup, StatusCallback, webhook de Twilio, mulaw, stream de audio, Twilio REST
  API, twilio-python, twilio-node, o cualquier concepto del ecosistema Twilio.
  También activar cuando el usuario tenga problemas con llamadas que se caen,
  AMD que no detecta buzón, TwiML que no funciona, Media Streams que se
  desconectan, o webhooks que no llegan. Esta skill detecta configuraciones
  incorrectas, antipatrones y bugs en integraciones Twilio con ejemplos
  corregidos listos para producción.
---
metadata:
  author: https://github.com/favelasquez

# Twilio � Skill Experta Completa

## Productos cubiertos

| Producto | Descripción | Referencia |
|---|---|---|
| **Voice + TwiML** | Control de flujo de llamadas | `references/twiml-voice.md` |
| **Media Streams** | Audio bidireccional en tiempo real | `references/media-streams.md` |
| **AMD** | Detección de buzón de voz | `references/amd.md` |
| **SMS / WhatsApp** | Mensajería | `references/sms-whatsapp.md` |
| **Webhooks y callbacks** | Eventos, errores, estado | `references/webhooks.md` |

---
metadata:
  author: https://github.com/favelasquez

## Paso 1 � Diagnóstico rápido

```
¿Qué tipo de llamada?
  Entrante  �  Twilio llama a tu webhook con el CallSid �  tu servidor responde TwiML
  Saliente  �  Tu servidor llama a Twilio REST API �  Twilio llama al número destino

¿Cuál es el problema?
  Llamada se cae sola          �  TwiML sin <Pause> suficiente o WebSocket sin ping
  AMD no detecta buzón         �  machine_detection_timeout corto o DetectMessageEnd mal configurado
  Audio corrupto               �  encoding/sample_rate incorrecto en Media Streams
  Webhook no llega             �  URL no pública, falta HTTPS, o ngrok expiró
  Media Stream se desconecta   �  Sin keep-alive o servidor WebSocket sin ping_interval
  TwiML no hace nada           �  Content-Type debe ser application/xml, no text/html
  "Cita confirmada" pero no cuelga �  Falta <Hangup/> o no se llama calls(sid).update()

¿Qué SDK usas?
  from twilio.rest import Client     �  Python (twilio>=8.0)
  const twilio = require('twilio')   �  Node.js
  TwiML via REST API                 �  cualquier lenguaje con HTTP
```

---
metadata:
  author: https://github.com/favelasquez

## Flujo de llamada saliente con Virtual Agent � mapa completo

```
Tu servidor
    �
    �S�� 1. calls.create(to, from, url, machine_detection) �����  Twilio REST API
    �                                                              �
    �  � ����� CallSid devuelto inmediatamente                        �
    �                                                          Twilio marca
    �                                                              �
    �  � ����� POST /twiml/outbound-hold (TwiML inicial) � ������������� Teléfono contesta
    �      Respuesta: <Pause length="60"/>                         �
    �                                                          AMD trabajando...
    �  � ����� POST /amd-result (AnsweredBy=human/machine_end_beep)    �
    �      �  Si humano: update(twiml con <Stream>)                 �
    �      �  Si buzón:  update(twiml con <Say>+<Hangup/>)          �
    �                                                              �
    �  � ����� WebSocket /ws/agent � ��������������������������������������������� Media Stream abierto
    �      Audio μ-law 8kHz bidireccional                          �
    �                                                              �
    �  � ����� POST /call-status (completed/failed/busy)          Llamada termina
```

---
metadata:
  author: https://github.com/favelasquez

## Checklist de auditoría � ejecutar en cada revisión

### �x� Llamadas y TwiML

- [ ] ¿El webhook de Twilio responde con `Content-Type: application/xml`?
- [ ] ¿El TwiML de llamadas salientes tiene `<Pause length="60"/>` para mantener la llamada?
- [ ] ¿Se usa `<Hangup/>` para colgar o `calls(sid).update()` desde el servidor?
- [ ] ¿Las URLs de webhook son HTTPS y públicas? (no localhost)
- [ ] ¿El StatusCallback está configurado para recibir eventos de la llamada?

### �x� AMD (Answering Machine Detection)

- [ ] ¿Se usa `machine_detection="DetectMessageEnd"` (no solo `"Enable"`)?
- [ ] ¿`machine_detection_timeout` es al menos 45 segundos?
- [ ] ¿`async_amd=True` para no bloquear la llamada mientras detecta?
- [ ] ¿El callback AMD maneja `human`, `machine_end_beep`, `machine_end_silence`, `machine_end_other`, `unknown` y `fax`?
- [ ] ¿`unknown` tiene fallback (tratar como humano)?

### �x� Media Streams

- [ ] ¿El WebSocket del servidor tiene `ping_interval` configurado?
- [ ] ¿El audio enviado de vuelta a Twilio está en formato μ-law 8kHz?
- [ ] ¿Se envía `{"event":"clear","streamSid":sid}` al interrumpir el TTS (barge-in)?
- [ ] ¿Se manejan los eventos `connected`, `start`, `media`, `stop` de Twilio?
- [ ] ¿El audio de Twilio se decodifica de Base64 antes de procesarlo?
- [ ] ¿El `streamSid` se captura del evento `start` para usarlo en respuestas?

### �xx� Webhooks y seguridad

- [ ] ¿Se valida la firma de Twilio en los webhooks? (`X-Twilio-Signature`)
- [ ] ¿Las credenciales están en variables de entorno, no en código?
- [ ] ¿Los webhooks responden en menos de 15 segundos? (timeout de Twilio)
- [ ] ¿Hay retry logic si el webhook falla?

---
metadata:
  author: https://github.com/favelasquez

## Bugs más comunes � referencia rápida

### 1. Content-Type incorrecto � TwiML ignorado silenciosamente

```python
# �R Twilio recibe HTML/JSON en lugar de XML �  cuelga la llamada
@app.post("/webhook")
async def webhook():
    return {"twiml": "<Response><Say>Hola</Say></Response>"}  # JSON � incorrecto

# �S& Siempre application/xml
from fastapi.responses import Response

@app.post("/webhook")
async def webhook():
    twiml = """<?xml version="1.0" encoding="UTF-8"?>
<Response>
    <Say voice="Polly.Lupe" language="es-US">Hola, ¿cómo puedo ayudarte?</Say>
    <Pause length="60"/>
</Response>"""
    return Response(content=twiml, media_type="application/xml")
```

### 2. Llamada se cae inmediatamente � TwiML sin pausa

```xml
<!-- �R Twilio ejecuta el Say y cuelga � sin tiempo para conversación -->
<Response>
    <Start><Stream url="wss://servidor.com/ws"/></Start>
    <Say>Hola, estoy aquí para ayudarte</Say>
</Response>
<!-- Cuando termina el <Say>, Twilio cuelga �  WebSocket se cierra -->

<!-- �S& <Pause> para mantener la llamada abierta -->
<Response>
    <Start><Stream url="wss://servidor.com/ws"/></Start>
    <Pause length="120"/>
</Response>
<!-- 120s de pausa �  la llamada dura hasta que el servidor la cuelgue -->
```

### 3. Colgar desde el servidor � forma correcta

```python
# �R Solo cerrar el WebSocket � Twilio puede tardar 5s en detectarlo
await websocket.close()

# �R Responder TwiML con Hangup � solo funciona en respuesta directa al webhook
# No funciona para colgar desde el servidor en cualquier momento

# �S& Llamar a la REST API � instantáneo
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

### 4. Media Stream � audio enviado antes de que abra

```python
# �R El evento 'start' aún no llegó � stream_sid desconocido
async def send_audio_to_twilio(ws, audio_bytes: bytes):
    await ws.send(json.dumps({
        "event": "media",
        "streamSid": None,  # � � None porque 'start' no llegó aún
        "media": {"payload": base64.b64encode(audio_bytes).decode()}
    }))

# �S& Capturar stream_sid en el evento 'start' antes de enviar audio
stream_sid = None

async for message in twilio_ws:
    packet = json.loads(message)
    event = packet.get("event")

    if event == "start":
        stream_sid = packet["start"]["streamSid"]
        print(f"Stream abierto: {stream_sid}")

    elif event == "media" and stream_sid:
        # Ahora sí tenemos el streamSid � procesar y responder
        audio = base64.b64decode(packet["media"]["payload"])
        # ... procesar audio ...

    elif event == "stop":
        stream_sid = None
        break
```

### 5. Barge-in � vaciar buffer de Twilio

```python
# �R Cancelar el TTS pero el audio ya enviado sigue sonando en Twilio
tts_task.cancel()  # cancela la generación, pero Twilio ya tiene audio en buffer

# �S& Enviar 'clear' + cancelar el task
async def interrupt_agent(twilio_ws, stream_sid: str, tts_task: asyncio.Task):
    # 1. Vaciar el buffer de audio de Twilio
    await twilio_ws.send(json.dumps({
        "event": "clear",
        "streamSid": stream_sid
    }))
    # 2. Cancelar la generación de TTS
    if tts_task and not tts_task.done():
        tts_task.cancel()
    print("Buffer vaciado + TTS cancelado")
```

### 6. Validar firma de Twilio � webhooks falsos

```python
# �R Sin validación � cualquiera puede enviar requests a tu webhook
@app.post("/webhook")
async def webhook(request: Request):
    form = await request.form()
    # procesar sin verificar que viene de Twilio

# �S& Validar X-Twilio-Signature
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
        raise HTTPException(status_code=403, detail="Firma inválida")
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

Leer archivos `references/` para guías completas por área.

