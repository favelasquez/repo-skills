---
metadata:
  author: https://github.com/favelasquez
name: deepgram-expert
description: >
  Skill experta en Deepgram cubriendo Speech-to-Text en tiempo real (Live/Streaming),
  STT por archivo (Pre-recorded), Text-to-Speech (TTS/Aura) y Audio Intelligence.
  Activar siempre que el usuario mencione Deepgram, transcripciÃ³n de audio, WebSockets
  de audio, Twilio + transcripciÃ³n, STT, TTS, mulaw, PCM, audio encoding, sample rate,
  diarizaciÃ³n, punctuate, interim_results, is_final, UtteranceEnd, keep-alive de
  Deepgram, modelos nova-2, whisper, enhanced, reconexiÃ³n de WebSocket, SDK de Deepgram
  en Python, o cualquier concepto relacionado con procesamiento de voz en tiempo real.
  Esta skill detecta configuraciones incorrectas de audio, bugs de conexiÃ³n, parÃ¡metros
  mal configurados y antipatrones en el manejo de transcripciones, con ejemplos
  corregidos para producciÃ³n.
---
metadata:
  author: https://github.com/favelasquez

# Deepgram â€” Skill Experta Completa

## Productos cubiertos

| Producto | Uso | Referencia |
|---|---|---|
| **Live Streaming** | STT en tiempo real vÃ­a WebSocket | `references/live-streaming.md` |
| **Pre-recorded** | STT por archivo o URL | `references/pre-recorded.md` |
| **TTS / Aura** | Text-to-Speech | `references/tts-aura.md` |
| **Audio Intelligence** | Summarize, Topics, Sentiment, Diarization | `references/audio-intelligence.md` |
| **Integraciones** | Twilio, WebSockets, Python SDK, REST | `references/integraciones.md` |

---
metadata:
  author: https://github.com/favelasquez

## Paso 1 â€” DiagnÃ³stico rÃ¡pido del problema

Antes de revisar cualquier cÃ³digo, identificar:

```
Â¿QuÃ© tipo de audio viene?
  Twilio / telefonÃ­a   â†’ mulaw (Î¼-law), 8000 Hz, mono â€” SIEMPRE
  Navegador (browser)  â†’ opus o pcm, 16000-48000 Hz
  MicrÃ³fono directo    â†’ pcm_s16le, 16000 Hz recomendado
  Archivo mp3/wav/ogg  â†’ Pre-recorded API, no Live

Â¿QuÃ© versiÃ³n del SDK de Python?
  from deepgram import Deepgram          â†’ SDK v1 (legacy)
  from deepgram import DeepgramClient    â†’ SDK v3 (actual, recomendado)

Â¿QuÃ© seÃ±ales de error hay?
  TranscripciÃ³n vacÃ­a o basura       â†’ encoding/sample_rate incorrecto
  ConexiÃ³n se cierra sola            â†’ falta keep-alive o ping_interval
  "is_final nunca llega"             â†’ falta UtteranceEnd o endpointing
  Lag alto en tiempo real            â†’ model incorrecto o no streaming
  "Cannot read property of undefined"â†’ response no parseada como JSON
```

---
metadata:
  author: https://github.com/favelasquez

## Configuraciones de audio por fuente â€” referencia rÃ¡pida

```python
# â”€â”€â”€ Twilio (llamadas telefÃ³nicas) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
TWILIO_CONFIG = {
    'encoding':     'mulaw',           # G.711 Î¼-law â€” OBLIGATORIO para Twilio
    'sample_rate':  8000,              # 8kHz â€” SIEMPRE en telefonÃ­a Twilio
    'channels':     1,                 # mono
    'model':        'nova-2-phonecall',# modelo optimizado para llamadas
    'punctuate':    True,
    'interim_results': True,
    'endpointing':  300,               # ms de silencio para detectar fin de frase
    'utterance_end_ms': '1000',        # ms extra para confirmar utterance
}

# â”€â”€â”€ Navegador / WebRTC â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
BROWSER_CONFIG = {
    'encoding':     'opus',            # WebRTC usa Opus por defecto
    'sample_rate':  48000,             # 48kHz en la mayorÃ­a de navegadores
    'channels':     1,
    'model':        'nova-2',
    'punctuate':    True,
    'interim_results': True,
    'smart_format': True,              # formatea nÃºmeros, fechas, etc.
}

# â”€â”€â”€ MicrÃ³fono Python (PyAudio) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
MICROPHONE_CONFIG = {
    'encoding':     'linear16',        # PCM raw 16-bit
    'sample_rate':  16000,             # 16kHz â€” balance calidad/latencia
    'channels':     1,
    'model':        'nova-2',
    'punctuate':    True,
    'interim_results': True,
}

# â”€â”€â”€ Archivo de audio (Pre-recorded) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
PRERECORDED_CONFIG = {
    'model':          'nova-2',
    'punctuate':      True,
    'diarize':        True,            # identificar quiÃ©n habla
    'smart_format':   True,
    'paragraphs':     True,
    'summarize':      'v2',
    'detect_language': True,
}
```

---
metadata:
  author: https://github.com/favelasquez

## Checklist de auditorÃ­a â€” ejecutar en cada revisiÃ³n

### ðŸ”´ ConfiguraciÃ³n de audio

- [ ] Â¿El `encoding` coincide con el formato real del audio? (mulaw para Twilio, opus para browser)
- [ ] Â¿El `sample_rate` es correcto? (8000 para Twilio/telefonÃ­a, 16000-48000 para otros)
- [ ] Â¿Se usa `model: nova-2-phonecall` para llamadas telefÃ³nicas?
- [ ] Â¿El audio de Twilio se decodifica de Base64 a bytes antes de enviarlo?
- [ ] Â¿El SDK v1 vs v3 estÃ¡ siendo usado correctamente? (APIs distintas)

### ðŸ”´ ConexiÃ³n y ciclo de vida

- [ ] Â¿La conexiÃ³n tiene keep-alive para silencios >10 segundos?
- [ ] Â¿Se maneja `ConnectionClosed` y se reconecta automÃ¡ticamente?
- [ ] Â¿Se llama `finish()` / `close()` al terminar para flush del buffer?
- [ ] Â¿El evento `stop` de Twilio cierra la conexiÃ³n con Deepgram?
- [ ] Â¿Hay timeout configurado para detectar conexiones muertas?

### ðŸ”´ Procesamiento de respuestas

- [ ] Â¿Las respuestas se parsean como JSON antes de usar?
- [ ] Â¿Se filtra por `is_final: true` para obtener solo transcripciones completas?
- [ ] Â¿Se maneja `UtteranceEnd` para detectar fin de turno del hablante?
- [ ] Â¿Se extrae correctamente de `channel.alternatives[0].transcript`?
- [ ] Â¿Se ignoran respuestas con `transcript` vacÃ­o (segmentos de silencio)?

### ðŸŸ  Seguridad y arquitectura

- [ ] Â¿La API key viene de variable de entorno, no hardcodeada?
- [ ] Â¿En browser/frontend, el WebSocket de Deepgram pasa por backend? (nunca exponer key al cliente)
- [ ] Â¿Los errores de Deepgram se logean y manejan, no solo se imprimen?

---
metadata:
  author: https://github.com/favelasquez

## Bugs mÃ¡s comunes â€” referencia rÃ¡pida

### 1. Encoding incorrecto â€” el bug #1 de Twilio + Deepgram

```python
# âŒ Deepgram asume PCM 16kHz â€” recibe Î¼-law 8kHz â†’ transcripciÃ³n basura
dg_connection = await client.transcription.live({
    'punctuate': True,
})

# âœ… SIEMPRE especificar encoding y sample_rate para Twilio
dg_connection = await client.transcription.live({
    'encoding':    'mulaw',
    'sample_rate': 8000,
    'channels':    1,
    'model':       'nova-2-phonecall',
    'punctuate':   True,
    'interim_results': True,
})
```

### 2. Base64 sin decodificar â€” audio corrupto

```python
# âŒ Twilio envÃ­a el audio como Base64 en el JSON â€” enviarlo directo es texto, no audio
payload = packet['media']['payload']    # string: "SUQzBAAAAAAAI..."
dg_connection.send(payload)             # Deepgram recibe texto, no audio

# âœ… Decodificar primero
import base64
audio_bytes: bytes = base64.b64decode(packet['media']['payload'])
await dg_connection.send(audio_bytes)
```

### 3. SDK v1 vs v3 â€” APIs completamente distintas

```python
# â”€â”€â”€ SDK v1 (legacy) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
from deepgram import Deepgram
client = Deepgram(API_KEY)
connection = await client.transcription.live(options)
connection.send(audio_bytes)           # sÃ­ncrono en v1
await connection.finish()

# â”€â”€â”€ SDK v3 (actual â€” recomendado) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
from deepgram import DeepgramClient, LiveTranscriptionEvents, LiveOptions
client = DeepgramClient(API_KEY)
dg_connection = client.listen.asynclive.v("1")

# Callbacks en lugar de recv()
async def on_transcript(self, result, **kwargs):
    if result.is_final:
        transcript = result.channel.alternatives[0].transcript
        if transcript:
            print(transcript)

dg_connection.on(LiveTranscriptionEvents.Transcript, on_transcript)
await dg_connection.start(LiveOptions(
    encoding="mulaw",
    sample_rate=8000,
    model="nova-2-phonecall",
))
```

### 4. `is_final` no filtrado â€” spam de parciales

```python
# âŒ Con interim_results=True, Deepgram envÃ­a actualizaciÃ³n por cada palabra
# "hol" â†’ "hola" â†’ "hola co" â†’ "hola como" â†’ "hola como estÃ¡s" (5 eventos, 1 frase)
data = json.loads(response)
print(data['channel']['alternatives'][0]['transcript'])  # imprime todos

# âœ… Filtrar solo los finales
data = json.loads(response)
if not data.get('is_final'):
    return  # ignorar parciales â€” esperar el definitivo
transcript = data['channel']['alternatives'][0]['transcript'].strip()
if transcript:
    print(transcript)
```

### 5. Sin keep-alive â€” conexiÃ³n muere en silencios

```python
# âŒ Deepgram cierra la conexiÃ³n si no recibe audio por ~10-12 segundos
# Pausas largas, mÃºsica en espera, usuario pensando â†’ transcripciÃ³n se corta

# âœ… OPCIÃ“N A â€” enviar silencio Î¼-law periÃ³dicamente
async def keepalive_loop(dg_connection, stop_event: asyncio.Event):
    silence = b'\x7f' * 800  # 100ms de silencio Î¼-law (8000Hz Ã— 0.1s)
    while not stop_event.is_set():
        await asyncio.sleep(8)  # cada 8s â€” antes del lÃ­mite de 10s
        try:
            await dg_connection.send(silence)
        except Exception:
            break

# âœ… OPCIÃ“N B â€” KeepAlive message (SDK v3)
await dg_connection.keep_alive()
```

### 6. Sin manejo de `UtteranceEnd` â€” fin de turno perdido

```python
# âŒ Solo con is_final no se sabe cuÃ¡ndo el usuario TERMINÃ“ su turno
# (vs una pausa breve dentro de una frase larga)

# âœ… Usar utterance_end_ms + evento UtteranceEnd (SDK v3)
options = LiveOptions(
    encoding="mulaw",
    sample_rate=8000,
    model="nova-2-phonecall",
    interim_results=True,
    utterance_end_ms="1000",  # 1s de silencio = fin de turno
    vad_events=True,          # Voice Activity Detection
)

async def on_utterance_end(self, utterance_end, **kwargs):
    # El usuario terminÃ³ de hablar â€” procesar turno completo
    print("Turno terminado â€” procesar respuesta")
    await process_turn(accumulated_transcript)

dg_connection.on(LiveTranscriptionEvents.UtteranceEnd, on_utterance_end)
```

---
metadata:
  author: https://github.com/favelasquez

## Modelos disponibles â€” cuÃ¡ndo usar cada uno

| Modelo | Caso de uso | Latencia | PrecisiÃ³n |
|---|---|---|---|
| `nova-2` | General, mejor opciÃ³n por defecto | Baja | â­â­â­â­â­ |
| `nova-2-phonecall` | Twilio, llamadas telefÃ³nicas, 8kHz | Baja | â­â­â­â­â­ para voz |
| `nova-2-meeting` | Reuniones, mÃºltiples hablantes | Baja | â­â­â­â­â­ para meetings |
| `nova-2-voicemail` | Mensajes de voz grabados | Media | â­â­â­â­â­ para VM |
| `enhanced` | PrecisiÃ³n mÃ¡xima, sin urgencia | Alta | â­â­â­â­â­ |
| `base` | EconÃ³mico, velocidad sobre precisiÃ³n | Muy baja | â­â­â­ |
| `whisper-large` | Compatibilidad OpenAI Whisper | Alta | â­â­â­â­ |

**Regla:** Para Twilio siempre `nova-2-phonecall`. Para todo lo demÃ¡s, `nova-2`.


---
metadata:
  author: https://github.com/favelasquez

## Virtual Agent con Twilio â€” Perfil de fallas configurado

**Fallas activas identificadas:**
- ðŸ”´ Agente no para cuando el usuario habla encima (**barge-in ausente**)
- ðŸ”´ BuzÃ³n de voz no se detecta o el agente trata de hablar con la mÃ¡quina
- ðŸ”´ Colgar no es limpio â€” la llamada queda abierta o cuelga abruptamente
- ðŸ”´ Agente no detecta cuÃ¡ndo el usuario terminÃ³ su turno
- ðŸ”´ Lag alto entre que el usuario habla y el agente responde

**Stack:** Twilio Media Streams â†’ Deepgram STT â†’ LLM â†’ TTS â†’ Twilio

> Referencia especializada: `references/virtual-agent.md`

### DiagnÃ³stico rÃ¡pido de cada falla

```
Agente sigue hablando cuando el usuario interrumpe (barge-in)
  â†’ Causa #1: No hay cancelaciÃ³n del task de TTS cuando VAD detecta voz del usuario
  â†’ Causa #2: No se envÃ­a {"event":"clear"} a Twilio â€” el buffer de audio ya enviado sigue sonando
  â†’ Causa #3: vad_events=True no estÃ¡ en la config de Deepgram â†’ SpeechStarted nunca llega
  â†’ SoluciÃ³n: BargeInController que cancela el asyncio.Task del TTS + vacÃ­a buffer de Twilio

Agente no detecta fin de turno del usuario
  â†’ Causa #1: Falta utterance_end_ms en la config de Deepgram
  â†’ Causa #2: Falta vad_events=True
  â†’ Causa #3: endpointing muy alto (>500ms) â€” espera demasiado
  â†’ Causa #4: LÃ³gica basada solo en is_final sin acumular el turno completo

BuzÃ³n de voz no se detecta / agente habla con la mÃ¡quina
  â†’ Causa #1: No se usa machine_detection="DetectMessageEnd" en la llamada saliente
  â†’ Causa #2: AMD callback no maneja "machine_end_beep" â€” el momento exacto post-beep
  â†’ Causa #3: machine_detection_timeout demasiado corto â€” buzones en espaÃ±ol tardan mÃ¡s
  â†’ Causa #4: AnsweredBy="unknown" no tiene fallback â€” se pierde la llamada
  â†’ SoluciÃ³n: AMD completo con callback que distingue humano/buzÃ³n/fax/unknown

Colgar limpiamente
  â†’ Causa #1: No se llama twilio_client.calls(sid).update(twiml="<Hangup/>")
  â†’ Causa #2: Solo se cierra el WebSocket â€” Twilio tarda hasta 5s en detectarlo
  â†’ Causa #3: No hay detecciÃ³n de frases de despedida en la respuesta del LLM
  â†’ SoluciÃ³n: hang_up_call() con API de Twilio + lista de HANGUP_SIGNALS

Lag alto (>2s de delay)
  â†’ Causa #1: LLM llamado de forma sÃ­ncrona bloqueando el event loop
  â†’ Causa #2: TTS generado completo antes de empezar a reproducir (no streaming)
  â†’ Causa #3: Acumulando demasiados tokens antes de invocar al LLM
  â†’ Causa #4: modelo equivocado â€” usar nova-2-phonecall, no enhanced
```

### Checklist especÃ­fico para Virtual Agent

- [ ] Â¿La config de Deepgram tiene `utterance_end_ms`, `vad_events` y `endpointing`?
- [ ] Â¿El keep-alive se envÃ­a cada 8s cuando no hay audio del usuario?
- [ ] Â¿El `asyncio.gather()` usa `return_exceptions=True` para que un task no mate a los demÃ¡s?
- [ ] Â¿El TwiML tiene `<Pause length="60"/>` suficiente para que dure la conversaciÃ³n?
- [ ] Â¿La llamada saliente usa `machine_detection='DetectMessageEnd'` para AMD?
- [ ] Â¿Hay un callback de AMD que desvÃ­a al flujo de buzÃ³n cuando se detecta mÃ¡quina?
- [ ] Â¿El LLM se invoca con `await` y no bloquea el event loop?
- [ ] Â¿El TTS se reproduce en streaming (chunks) y no espera el audio completo?
- [ ] Â¿La lÃ³gica acumula el turno completo antes de invocar al LLM?
- [ ] Â¿El estado de la conversaciÃ³n persiste entre turnos (contexto del LLM)?

---
metadata:
  author: https://github.com/favelasquez

Leer archivos `references/` para guÃ­as completas por Ã¡rea.

