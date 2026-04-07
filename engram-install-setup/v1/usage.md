---
metadata:
  version: "1.0"
  author: https://github.com/favelasquez
name: engram-install-setup
description: >
  Skill para instalar, configurar y verificar Engram en la máquina del usuario.
  Activar cuando el usuario quiera instalar Engram, configurarlo en Claude Code,
  Cursor, Windsurf, VS Code Copilot, OpenCode o Gemini CLI, cuando pregunte cómo
  empezar con Engram, cuando tenga problemas de instalación, cuando quiera configurar
  Engram como servicio en background, o cuando diga "instala Engram", "configura
  Engram en mi agente", "engram no funciona", "cómo conecto Engram con Claude Code".
  Esta skill ejecuta los pasos de instalación reales, verifica que funcionen, y deja
  el sistema listo para usar.
---
metadata:
  author: https://github.com/favelasquez

# Engram Install & Setup � Instalación y configuración guiada

## Paso 1 � Detectar el sistema antes de actuar

```bash
# Detectar OS y herramientas disponibles
uname -s                    # Darwin=macOS, Linux=Linux
which brew 2>/dev/null      # ¿tiene Homebrew?
which go 2>/dev/null        # ¿tiene Go?
which engram 2>/dev/null    # ¿ya está instalado?
echo $PATH                  # verificar que ~/.local/bin esté en PATH
```

---
metadata:
  author: https://github.com/favelasquez

## Paso 2 � Instalar el binario

### macOS (Homebrew � recomendado)
```bash
brew install gentleman-programming/tap/engram
```

### Linux / macOS sin Homebrew (desde fuente)
```bash
# Requiere Go 1.21+
git clone https://github.com/Gentleman-Programming/engram.git
cd engram
go build -o engram ./cmd/engram

# Mover a PATH
mkdir -p ~/.local/bin
mv engram ~/.local/bin/engram

# Verificar que ~/.local/bin está en PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc   # o ~/.bashrc
source ~/.zshrc
```

### Verificar instalación
```bash
engram version
# Esperado: engram vX.X.X
```

---
metadata:
  author: https://github.com/favelasquez

## Paso 3 � Setup por agente

### Claude Code
```bash
claude plugin marketplace add Gentleman-Programming/engram
claude plugin install engram
# El plugin inyecta automáticamente el Memory Protocol en el system prompt
# No necesita configuración adicional
```

### OpenCode
```bash
engram setup opencode
```

### Gemini CLI
```bash
engram setup gemini-cli
```

### VS Code Copilot
```bash
code --add-mcp '{"name":"engram","command":"engram","args":["mcp"]}'
```

### Cursor / Windsurf / cualquier agente con MCP manual
Agregar a la configuración MCP del agente:
```json
{
  "mcp": {
    "engram": {
      "type": "stdio",
      "command": "engram",
      "args": ["mcp"]
    }
  }
}
```

**En Cursor:** Settings �  MCP �  Add Server �  pegar el JSON de arriba
**En Windsurf:** ~/.codeium/windsurf/mcp_config.json

---
metadata:
  author: https://github.com/favelasquez

## Paso 4 � Iniciar el servidor HTTP (opcional pero recomendado)

```bash
# Probar que funciona
engram serve &
curl http://localhost:7437/health
# Esperado: {"status":"ok"}
```

### Configurar como servicio permanente (Linux systemd)
```bash
mkdir -p ~/.engram ~/.config/systemd/user

cat > ~/.config/systemd/user/engram.service << 'EOF'
[Unit]
Description=Engram Memory Server
After=network.target

[Service]
WorkingDirectory=%h
ExecStart=%h/.local/bin/engram serve
Restart=always
RestartSec=3
Environment=ENGRAM_DATA_DIR=%h/.engram

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable engram
systemctl --user start engram
systemctl --user status engram   # verificar
```

### macOS � LaunchAgent (background permanente)
```bash
mkdir -p ~/Library/LaunchAgents

cat > ~/Library/LaunchAgents/com.engram.serve.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.engram.serve</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/engram</string>
    <string>serve</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>StandardOutPath</key>
  <string>/tmp/engram.log</string>
  <key>StandardErrorPath</key>
  <string>/tmp/engram-error.log</string>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/com.engram.serve.plist
# Verificar:
curl http://localhost:7437/health
```

---
metadata:
  author: https://github.com/favelasquez

## Paso 5 � Verificación completa

```bash
# 1. Binario accesible
engram version

# 2. Servidor HTTP funcionando
curl -s http://localhost:7437/health

# 3. MCP server responde
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | engram mcp | head -5

# 4. Guardar una memoria de prueba
engram save "Setup Engram completado" \
  "**What**: Engram instalado y configurado\n**Why**: Memoria persistente para el agente\n**Where**: Sistema local\n**Learned**: Usar engram tui para explorar memorias" \
  --type config

# 5. Buscar la memoria de prueba
engram search "Setup Engram"

# 6. Abrir TUI para explorar (opcional)
engram tui
```

---
metadata:
  author: https://github.com/favelasquez

## Troubleshooting de instalación

### `engram: command not found`
```bash
# Verificar dónde está el binario
find ~ -name "engram" -type f 2>/dev/null

# Añadir al PATH
echo 'export PATH="$HOME/.local/bin:$HOME/go/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### El plugin de Claude Code no carga
```bash
# Reinstalar
claude plugin uninstall engram 2>/dev/null
claude plugin marketplace add Gentleman-Programming/engram
claude plugin install engram
# Reiniciar Claude Code completamente
```

### Puerto 7437 ya en uso
```bash
lsof -i :7437                    # ver qué proceso lo usa
# Usar otro puerto:
ENGRAM_PORT=7438 engram serve
# Actualizar la config del agente con el nuevo puerto
```

### Error de permisos en macOS
```bash
# Si Gatekeeper bloquea el binario:
xattr -dr com.apple.quarantine $(which engram)
```

### Engram no recuerda nada después de reiniciar
```bash
# Verificar dónde guarda los datos
ls ~/.engram/
# Si está vacío, el ENGRAM_DATA_DIR puede ser diferente
echo $ENGRAM_DATA_DIR
# Asegurarse de que el servicio usa el mismo directorio siempre
```

---
metadata:
  author: https://github.com/favelasquez

## Variables de entorno

```bash
# En ~/.zshrc o ~/.bashrc
export ENGRAM_DATA_DIR=~/.engram    # directorio de datos (default)
export ENGRAM_PORT=7437             # puerto HTTP (default)
```

---
metadata:
  author: https://github.com/favelasquez

## Checklist de instalación exitosa

- [ ] `engram version` devuelve una versión
- [ ] `curl http://localhost:7437/health` devuelve `{"status":"ok"}`
- [ ] `engram search "test"` no da error
- [ ] El agente (Claude Code/Cursor/etc.) muestra Engram en sus tools MCP
- [ ] `engram tui` abre la interfaz correctamente

Cuando todo el checklist esté verde, Engram está listo para usar.

