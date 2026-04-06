---
name: deepgram-expert
description: >
  Skill experta en Deepgram cubriendo Speech-to-Text en tiempo real (Live/Streaming),
  STT por archivo (Pre-recorded), Text-to-Speech (TTS/Aura) y Audio Intelligence.
  Activar siempre que el usuario mencione Deepgram, transcripción de audio, WebSockets
  de audio, Twilio + transcripción, STT, TTS, mulaw, PCM, audio encoding, sample rate,
  diarización, punctuate, interim_results, is_final, UtteranceEnd, keep-alive de
  Deepgram, modelos nova-2, whisper, enhanced, reconexión de WebSocket, SDK de Deepgram
  en Python, o cualquier concepto relacionado con procesamiento de voz en tiempo real.
  Esta skill detecta configuraciones incorrectas de audio, bugs de conexión, parámetros
  mal configurados y antipatrones en el manejo de transcripciones, con ejemplos
  corregidos para producción.
---

# Deepgram — Skill Experta Completa

## Productos cubiertos

| Producto | Uso | Referencia |
|---|---|---|
| **Live Streaming** | STT en tiempo real vía WebSocket | `references/live-streaming.md` |
| **Pre-recorded** | STT por archivo o URL | `references/pre-recorded.md` |
| **TTS / Aura** | Text-to-Speech | `references/tts-aura.md` |
| **Audio Intelligence** | Summarize, Topics, Sentiment, Diarization | `references/audio-intelligence.md` |
| **Integraciones** | Twilio, WebSockets, Python SDK, REST | `references/integraciones.md` |

---

## Paso 1 — Diagnóstico rápido del problema

Antes de revisar cualquier código, identificar:

```
¿Qué tipo de audio viene?
  Twilio / telefonía   → mulaw (μ-law), 8000 Hz, mono — SIEMPRE
  Navegador (browser)  → opus o pcm, 16000-48000 Hz
  Micrófono directo    → pcm_s16le, 16000 Hz recomendado
  Archivo mp3/wav/ogg  → Pre-recorded API, no Live

¿Qué versión del SDK de Python?
  from deepgram import Deepgram          → SDK v1 (legacy)
  from deepgram import DeepgramClient    → SDK v3 (actual, recomendado)

¿Qué señales de error hay?
  Transcripción vacía o basura       → encoding/sample_rate incorrecto
  Conexión se cierra sola            → falta keep-alive o ping_interval
  "is_final nunca llega"             → falta UtteranceEnd o endpointing
  Lag alto en tiempo real            → model incorrecto o no streaming
  "Cannot read property of undefined"→ response no parseada como JSON
```

---

## Configuraciones de audio por fuente — referencia rápida

```python
# ─── Twilio (llamadas telefónicas) ────────────────────────────────────────
TWILIO_CONFIG = {
    'encoding':     'mulaw',           # G.711 μ-law — OBLIGATORIO para Twilio
    'sample_rate':  8000,              # 8kHz — SIEMPRE en telefonía Twilio
    'channels':     1,                 # mono
    'model':        'nova-2-phonecall',# modelo optimizado para llamadas
    'punctuate':    True,
    'interim_results': True,
    'endpointing':  300,               # ms de silencio para detectar fin de frase
    'utterance_end_ms': '1000',        # ms extra para confirmar utterance
}

# ─── Navegador / WebRTC ───────────────────────────────────────────────────
BROWSER_CONFIG = {
    'encoding':     'opus',            # WebRTC usa Opus por defecto
    'sample_rate':  48000,             # 48kHz en la mayoría de navegadores
    'channels':     1,
    'model':        'nova-2',
    'punctuate':    True,
    'interim_results': True,
    'smart_format': True,              # formatea números, fechas, etc.
}

# ─── Micrófono Python (PyAudio) ───────────────────────────────────────────
MICROPHONE_CONFIG = {
    'encoding':     'linear16',        # PCM raw 16-bit
    'sample_rate':  16000,             # 16kHz — balance calidad/latencia
    'channels':     1,
    'model':        'nova-2',
    'punctuate':    True,
    'interim_results': True,
}

# ─── Archivo de audio (Pre-recorded) ─────────────────────────────────────
PRERECORDED_CONFIG = {
    'model':          'nova-2',
    'punctuate':      True,
    'diarize':        True,            # identificar quién habla
    'smart_format':   True,
    'paragraphs':     True,
    'summarize':      'v2',
    'detect_language': True,
}
```

---

## Checklist de auditoría — ejecutar en cada revisión

### 🔴 Configuración de audio

- [ ] ¿El `encoding` coincide con el formato real del audio? (mulaw para Twilio, opus para browser)
- [ ] ¿El `sample_rate` es correcto? (8000 para Twilio/telefonía, 16000-48000 para otros)
- [ ] ¿Se usa `model: nova-2-phonecall` para llamadas telefónicas?
- [ ] ¿El audio de Twilio se decodifica de Base64 a bytes antes de enviarlo?
- [ ] ¿El SDK v1 vs v3 está siendo usado correctamente? (APIs distintas)

### 🔴 Conexión y ciclo de vida

- [ ] ¿La conexión tiene keep-alive para silencios >10 segundos?
- [ ] ¿Se maneja `ConnectionClosed` y se reconecta automáticamente?
- [ ] ¿Se llama `finish()` / `close()` al terminar para flush del buffer?
- [ ] ¿El evento `stop` de Twilio cierra la conexión con Deepgram?
- [ ] ¿Hay timeout configurado para detectar conexiones muertas?

### 🔴 Procesamiento de respuestas

- [ ] ¿Las respuestas se parsean como JSON antes de usar?
- [ ] ¿Se filtra por `is_final: true` para obtener solo transcripciones completas?
- [ ] ¿Se maneja `UtteranceEnd` para detectar fin de turno del hablante?
- [ ] ¿Se extrae correctamente de `channel.alternatives[0].transcript`?
- [ ] ¿Se ignoran respuestas con `transcript` vacío (segmentos de silencio)?

### 🟠 Seguridad y arquitectura

- [ ] ¿La API key viene de variable de entorno, no hardcodeada?
- [ ] ¿En browser/frontend, el WebSocket de Deepgram pasa por backend? (nunca exponer key al cliente)
- [ ] ¿Los errores de Deepgram se logean y manejan, no solo se imprimen?

---

## Bugs más comunes — referencia rápida

### 1. Encoding incorrecto — el bug #1 de Twilio + Deepgram

```python
# ❌ Deepgram asume PCM 16kHz — recibe μ-law 8kHz → transcripción basura
dg_connection = await client.transcription.live({
    'punctuate': True,
})

# ✅ SIEMPRE especificar encoding y sample_rate para Twilio
dg_connection = await client.transcription.live({
    'encoding':    'mulaw',
    'sample_rate': 8000,
    'channels':    1,
    'model':       'nova-2-phonecall',
    'punctuate':   True,
    'interim_results': True,
})
```

### 2. Base64 sin decodificar — audio corrupto

```python
# ❌ Twilio envía el audio como Base64 en el JSON — enviarlo directo es texto, no audio
payload = packet['media']['payload']    # string: "SUQzBAAAAAAAI..."
dg_connection.send(payload)             # Deepgram recibe texto, no audio

# ✅ Decodificar primero
import base64
audio_bytes: bytes = base64.b64decode(packet['media']['payload'])
await dg_connection.send(audio_bytes)
```

### 3. SDK v1 vs v3 — APIs completamente distintas

```python
# ─── SDK v1 (legacy) ──────────────────────────────────────────────────────
from deepgram import Deepgram
client = Deepgram(API_KEY)
connection = await client.transcription.live(options)
connection.send(audio_bytes)           # síncrono en v1
await connection.finish()

# ─── SDK v3 (actual — recomendado) ────────────────────────────────────────
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

### 4. `is_final` no filtrado — spam de parciales

```python
# ❌ Con interim_results=True, Deepgram envía actualización por cada palabra
# "hol" → "hola" → "hola co" → "hola como" → "hola como estás" (5 eventos, 1 frase)
data = json.loads(response)
print(data['channel']['alternatives'][0]['transcript'])  # imprime todos

# ✅ Filtrar solo los finales
data = json.loads(response)
if not data.get('is_final'):
    return  # ignorar parciales — esperar el definitivo
transcript = data['channel']['alternatives'][0]['transcript'].strip()
if transcript:
    print(transcript)
```

### 5. Sin keep-alive — conexión muere en silencios

```python
# ❌ Deepgram cierra la conexión si no recibe audio por ~10-12 segundos
# Pausas largas, música en espera, usuario pensando → transcripción se corta

# ✅ OPCIÓN A — enviar silencio μ-law periódicamente
async def keepalive_loop(dg_connection, stop_event: asyncio.Event):
    silence = b'\x7f' * 800  # 100ms de silencio μ-law (8000Hz × 0.1s)
    while not stop_event.is_set():
        await asyncio.sleep(8)  # cada 8s — antes del límite de 10s
        try:
            await dg_connection.send(silence)
        except Exception:
            break

# ✅ OPCIÓN B — KeepAlive message (SDK v3)
await dg_connection.keep_alive()
```

### 6. Sin manejo de `UtteranceEnd` — fin de turno perdido

```python
# ❌ Solo con is_final no se sabe cuándo el usuario TERMINÓ su turno
# (vs una pausa breve dentro de una frase larga)

# ✅ Usar utterance_end_ms + evento UtteranceEnd (SDK v3)
options = LiveOptions(
    encoding="mulaw",
    sample_rate=8000,
    model="nova-2-phonecall",
    interim_results=True,
    utterance_end_ms="1000",  # 1s de silencio = fin de turno
    vad_events=True,          # Voice Activity Detection
)

async def on_utterance_end(self, utterance_end, **kwargs):
    # El usuario terminó de hablar — procesar turno completo
    print("Turno terminado — procesar respuesta")
    await process_turn(accumulated_transcript)

dg_connection.on(LiveTranscriptionEvents.UtteranceEnd, on_utterance_end)
```

---

## Modelos disponibles — cuándo usar cada uno

| Modelo | Caso de uso | Latencia | Precisión |
|---|---|---|---|
| `nova-2` | General, mejor opción por defecto | Baja | ⭐⭐⭐⭐⭐ |
| `nova-2-phonecall` | Twilio, llamadas telefónicas, 8kHz | Baja | ⭐⭐⭐⭐⭐ para voz |
| `nova-2-meeting` | Reuniones, múltiples hablantes | Baja | ⭐⭐⭐⭐⭐ para meetings |
| `nova-2-voicemail` | Mensajes de voz grabados | Media | ⭐⭐⭐⭐⭐ para VM |
| `enhanced` | Precisión máxima, sin urgencia | Alta | ⭐⭐⭐⭐⭐ |
| `base` | Económico, velocidad sobre precisión | Muy baja | ⭐⭐⭐ |
| `whisper-large` | Compatibilidad OpenAI Whisper | Alta | ⭐⭐⭐⭐ |

**Regla:** Para Twilio siempre `nova-2-phonecall`. Para todo lo demás, `nova-2`.


---

## Virtual Agent con Twilio — Perfil de fallas configurado

**Fallas activas identificadas:**
- 🔴 Agente no para cuando el usuario habla encima (**barge-in ausente**)
- 🔴 Buzón de voz no se detecta o el agente trata de hablar con la máquina
- 🔴 Colgar no es limpio — la llamada queda abierta o cuelga abruptamente
- 🔴 Agente no detecta cuándo el usuario terminó su turno
- 🔴 Lag alto entre que el usuario habla y el agente responde

**Stack:** Twilio Media Streams → Deepgram STT → LLM → TTS → Twilio

> Referencia especializada: `references/virtual-agent.md`

### Diagnóstico rápido de cada falla

```
Agente sigue hablando cuando el usuario interrumpe (barge-in)
  → Causa #1: No hay cancelación del task de TTS cuando VAD detecta voz del usuario
  → Causa #2: No se envía {"event":"clear"} a Twilio — el buffer de audio ya enviado sigue sonando
  → Causa #3: vad_events=True no está en la config de Deepgram → SpeechStarted nunca llega
  → Solución: BargeInController que cancela el asyncio.Task del TTS + vacía buffer de Twilio

Agente no detecta fin de turno del usuario
  → Causa #1: Falta utterance_end_ms en la config de Deepgram
  → Causa #2: Falta vad_events=True
  → Causa #3: endpointing muy alto (>500ms) — espera demasiado
  → Causa #4: Lógica basada solo en is_final sin acumular el turno completo

Buzón de voz no se detecta / agente habla con la máquina
  → Causa #1: No se usa machine_detection="DetectMessageEnd" en la llamada saliente
  → Causa #2: AMD callback no maneja "machine_end_beep" — el momento exacto post-beep
  → Causa #3: machine_detection_timeout demasiado corto — buzones en español tardan más
  → Causa #4: AnsweredBy="unknown" no tiene fallback — se pierde la llamada
  → Solución: AMD completo con callback que distingue humano/buzón/fax/unknown

Colgar limpiamente
  → Causa #1: No se llama twilio_client.calls(sid).update(twiml="<Hangup/>")
  → Causa #2: Solo se cierra el WebSocket — Twilio tarda hasta 5s en detectarlo
  → Causa #3: No hay detección de frases de despedida en la respuesta del LLM
  → Solución: hang_up_call() con API de Twilio + lista de HANGUP_SIGNALS

Lag alto (>2s de delay)
  → Causa #1: LLM llamado de forma síncrona bloqueando el event loop
  → Causa #2: TTS generado completo antes de empezar a reproducir (no streaming)
  → Causa #3: Acumulando demasiados tokens antes de invocar al LLM
  → Causa #4: modelo equivocado — usar nova-2-phonecall, no enhanced
```

### Checklist específico para Virtual Agent

- [ ] ¿La config de Deepgram tiene `utterance_end_ms`, `vad_events` y `endpointing`?
- [ ] ¿El keep-alive se envía cada 8s cuando no hay audio del usuario?
- [ ] ¿El `asyncio.gather()` usa `return_exceptions=True` para que un task no mate a los demás?
- [ ] ¿El TwiML tiene `<Pause length="60"/>` suficiente para que dure la conversación?
- [ ] ¿La llamada saliente usa `machine_detection='DetectMessageEnd'` para AMD?
- [ ] ¿Hay un callback de AMD que desvía al flujo de buzón cuando se detecta máquina?
- [ ] ¿El LLM se invoca con `await` y no bloquea el event loop?
- [ ] ¿El TTS se reproduce en streaming (chunks) y no espera el audio completo?
- [ ] ¿La lógica acumula el turno completo antes de invocar al LLM?
- [ ] ¿El estado de la conversación persiste entre turnos (contexto del LLM)?

---

Leer archivos `references/` para guías completas por área.
