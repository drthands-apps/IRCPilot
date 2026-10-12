# IRCPilot - Comandos Adicionales para Convivencia, Usabilidad y Diversión

> **Documento de Extensión**: Propuestas de comandos que hacen IRCPilot más atractivo, usable y divertido sin comprometer los principios core.

---

## 🎯 Filosofía de Nuevos Comandos

- **No sobrecargar**: Cada comando debe resolver un problema real
- **Modular**: Cada canal puede habilitar/deshabilitar features
- **Descentralizado**: Los usuarios crean contenido, no el servidor
- **Divertido**: Internet fue divertido. Devolvemos eso.

---

## 🎮 Categoría 1: Juegos y Entretenimiento

### `GAME` - Sistema de Minijuegos

```
/game start <tipo> [apuesta_opcional]
/game join
/game leave
/game status
/game leaderboard
```

**Tipos de juegos:**

#### 1.1 Trivia Colaborativa
```
/game start trivia difficulty=medium category=history
  → Servidor: "TRIVIA STARTED: 5 preguntas, máx 30 segundos cada una"
  → Pregunta 1: "¿Quién fue el primer presidente de Argentina?"
  → Usuarios responden: /game answer "Juan Manuel de Rosas"
  → Servidor: "❌ Incorrecto. Respuesta: Manuel Belgrano"
  → Servidor: "⏱ Tiempo límite. Siguiente pregunta..."
  → Ganador: Usuario con más respuestas correctas
```

**Categorías**: Historia, Ciencia, Literatura, Películas, Anime, Videojuegos, Cultura general

**Propiedades:**
- Base de datos comunitaria (usuarios pueden aportar preguntas)
- Puntuación global acumulativa
- Leaderboard mensual/anual
- Preguntas de dificultad variable

---

#### 1.2 Adivinanzas y Retos
```
/game start riddles
  → Servidor publica una adivinanza
  → Usuarios compiten por adivinar
  → Primero correcto gana puntos
```

---

#### 1.3 Hangman (Ahorcado)
```
/game start hangman
  → Servidor: "_ _ _ _ _"
  → Usuario A: /game guess "a"
  → Servidor: "_ A _ _ _" (si es correcto)
  → Jugadores colaboran o compiten
```

---

#### 1.4 Dados y Apuestas (Opcional)
```
/game start dice [apuesta]
  → Usuarios apuestan puntos virtuales
  → Servidor tira dados criptográficamente seguros
  → Ganador se lleva la apuesta
```

**Nota**: Apuestas en puntos virtuales solo (sin dinero real).

---

#### 1.5 Uno / Memory / Rock-Paper-Scissors
```
/game start uno
  → 2-8 jugadores
  → Turnos automáticos
  → Chat integrado (puedes hablar mientras juegas)
```

---

### `LEADERBOARD` - Ranking Global

```
/leaderboard [type] [period]
/leaderboard trivia all_time
/leaderboard games this_month
/leaderboard messages this_week
```

**Tipos de ranking:**
- `trivia` - Mejores en trivia
- `games` - Más juegos ganados
- `messages` - Más activos (mensajes totales)
- `reputation` - Más queridos (votos positivos)
- `streaks` - Racha más larga (conectado cada día)
- `contributions` - Más aportes a la comunidad

**Salida:**
```
TOP 10 - TRIVIA (All Time)
  1. alice_fp (2430 points) ⭐
  2. bob_fp (2180 points)
  3. charlie_fp (1950 points)
  ...
  10. eve_fp (890 points)

YOU: #247 (340 points)

Next tier: #100 with 410 points (70 points to go!)
```

---

## 💬 Categoría 2: Comunicación Enriquecida

### `NICK ALIAS` - Múltiples Nicks Simultáneos

```
/nick add <nuevo_nick>
/nick remove <nick>
/nick switch <nick>
/nick list
```

**Caso de uso:**
- Usuario puede tener 3-5 nicks bajo misma identidad
- Cambia de nick para diferentes contextos (serio en #tech, casual en #gaming)
- Todos vinculados a misma identidad (para baneo/auditoría)
- Otros usuarios ven el cambio: "alice cambió nick a alice_gaming"

**Propiedades:**
- Cada nick puede tener diferentes `LANG_FILTER`, `VISIBILITY`
- Pero la identidad (fingerprint) es la misma
- Permite "separación de contextos" sin perder identidad

---

### `EMOTE` - Emojis y Reacciones Personalizadas

```
/emote add <nombre> <unicode> [descripción]
/emote use <nombre>
/emote list
/emote search <pattern>
```

**Ejemplo:**
```
/emote add clap 👏 "aplauso"
/emote add party 🎉 "celebración"
/emote add facepalm 🤦 "cara de vergüenza"

Usuario A: Acabo de terminar mi proyecto!
Usuario B: /emote use clap clap party

→ Servidor: "Usuario B: 👏 👏 🎉"
```

**Propiedades:**
- Base de datos comunitaria de emojis
- Cada usuario puede crear sus propios emojis
- Reacciones se cuentan y aparecen como "reactions"
- Compatible con shortcodes de Discord: `:clap:`, `:party:`

---

### `QUOTE` - Citas Memorizadas

```
/quote <nick> <mensaje_id>
/quote list [from_nick]
/quote random
```

**Ejemplo:**
```
Usuario A: "IRC es lo mejor que pasó en internet"
[Sistema asigna ID #12345 a este mensaje]

Usuario B: /quote A 12345
→ Servidor: "> Usuario A: IRC es lo mejor que pasó en internet"
```

**Propiedades:**
- Permite "contextualizar" mensajes previos
- Facilita responder a conversaciones largas
- Genera "hilo" sin estar en threading real
- Citas se archivan (historias hiladas)

---

### `THREAD` - Conversaciones Ramificadas (Opcional)

```
/thread start <mensaje_base>
/thread reply <thread_id> <respuesta>
/thread view <thread_id>
/thread collapse
```

**Caso de uso:**
- Usuario A plantea una pregunta
- Usuarios B, C, D responden en "hilo"
- El hilo no contamina el canal principal
- Otros usuarios pueden lurk o ignorar el hilo

---

### `VOICE` - Mensajes de Audio (Futuro)

```
/voice record <segundos_max>
/voice play <mensaje_id>
```

**Caso de uso:**
- Usuarios pueden grabar nota de voz
- Se almacena encriptado
- Otros usuarios pueden escuchar o descargar
- Alternativa a texto puro en dispositivos móviles

---

## 📚 Categoría 3: Conocimiento y Colaboración

### `WIKI` - Base de Conocimiento del Canal

```
/wiki create <título>
/wiki edit <page_id> <contenido>
/wiki link <page_id>
/wiki search <palabra_clave>
/wiki history <page_id>
```

**Ejemplo:**
```
Usuario A crea: /wiki create "Guía de Python para Principiantes"
→ Genera URL: https://ircpilot.local/wiki/python-guide-123

Usuario A: /wiki edit 123 "## Variables..."
Usuario B: /wiki edit 123 "## Funciones..." (edición colaborativa)

Otros usuarios: /wiki link 123
→ Genera referencias en el canal

Usuario C: /wiki search "python loops"
→ Encuentra todas las páginas con esas palabras
```

**Propiedades:**
- Versionado (ver quién cambió qué)
- Markdown soportado
- Acceso público o privado por canal
- Historial completo de ediciones

---

### `POLL` - Encuestas Rápidas

```
/poll create <pregunta> <opción1> <opción2> [opción3...]
/poll vote <poll_id> <opción>
/poll results <poll_id>
/poll close <poll_id>
```

**Ejemplo:**
```
Usuario A: /poll create "¿Dónde almorzar?" "Pizza" "Hamburguesa" "Sushi"
  Poll #999 created. Vote with: /poll vote 999 Pizza

Usuario B: /poll vote 999 Pizza
Usuario C: /poll vote 999 Sushi
Usuario D: /poll vote 999 Pizza

Usuario A: /poll results 999
  Pizza: 2 votos (66%)
  Sushi: 1 voto (33%)
  Hamburguesa: 0 votos
```

**Propiedades:**
- Votación en vivo
- Anonimato opcional
- Duración limitada (1 min a 1 semana)
- Resultados en tiempo real

---

### `BOOKMARK` - Guardar Mensajes Favoritos

```
/bookmark add <mensaje_id> [etiqueta]
/bookmark list [etiqueta]
/bookmark share <bookmark_id>
```

**Caso de uso:**
- Usuario encuentra mensaje genial
- Lo guarda para leerlo después
- Lo etiqueta (python, funny, idea)
- Puede compartir bookmark con otros

---

### `NOTE` - Notas Privadas del Servidor

```
/note create <nombre> <contenido>
/note edit <note_id> <contenido>
/note list
/note delete <note_id>
```

**Caso de uso:**
- Usuario toma nota en el servidor
- La nota se encripta y almacena
- Accesible desde cualquier cliente/dispositivo
- Perfecta para IRC móvil (notas "en la nube")

---

## 👥 Categoría 4: Comunidad y Eventos

### `EVENT` - Eventos Comunitarios

```
/event create <nombre> <fecha> <descripción>
/event join <event_id>
/event leave <event_id>
/event list [upcoming|past]
/event broadcast <event_id> <mensaje>
```

**Ejemplo:**
```
Usuario A: /event create "Game Night 2026-10-20" "Jugamos a Among Us a las 20:00"
  Event #42 created

Usuarios B, C, D: /event join 42
  Confirmación: "3 usuarios se unieron"

Día del evento:
  Organizador A: /event broadcast 42 "¡Empieza en 5 minutos!"
  → NOTICE to all joined: "Game Night empieza en 5 minutos"
  → Sistema crea canal temporal #event_42
```

**Propiedades:**
- Calendario compartido
- Recordatorios automáticos
- Conteo de asistentes
- Canales temporales para el evento

---

### `REPUTATION` - Sistema de Puntuación Social

```
/rep give <nick> [razón]
/rep check <nick|self>
/rep leaderboard
```

**Ejemplo:**
```
Usuario A ayuda a Usuario B con un problema
  Usuario B: /rep give A "excelente ayuda con Python"
  
Usuario A: /rep check
  Reputation: +47 (muy bueno)
  Recent: +3 en última semana
  
/rep leaderboard
  Top helpful users: alice (+250), bob (+180), charlie (+120)
```

**Propiedades:**
- Sistema de karma simplificado
- Votación positiva (no negativa, no "downvotes")
- Se ve en perfil público
- Afecta visibilidad en búsquedas
- Protege contra abusadores recurrentes

---

### `INTRODUCE` - Presentaciones Automáticas

```
/introduce <nombre> <intereses> <ubicación_opcional>
```

**Resultado:**
```
NUEVO USUARIO: alice_fp
  Intereses: Python, Gaming, Anime
  Ubicación: Argentina
  
¡Bienvenido al server! /help para comandos, /rules para reglas.
```

**Propiedades:**
- Presentación uniforme de nuevos usuarios
- Facilita conexiones por intereses
- Se archiva en perfil público (opt-in)

---

## 🛠️ Categoría 5: Utilidad y Productividad

### `REMINDER` - Recordatorios Personales

```
/reminder set <texto> <cuando>
/reminder list
/reminder delete <reminder_id>
```

**Ejemplo:**
```
/reminder set "Reunión con el equipo" "2026-10-20 15:00"
/reminder set "Revisar tarea" "in 2 hours"
/reminder set "Llamar a mamá" "tomorrow at 19:00"

→ Sistema guarda recordatorio
→ A la hora indicada: NOTICE "RECORDATORIO: Reunión con el equipo"
```

---

### `TRANSLATE` - Traducción Automática Opcional

```
/translate <nick> enable
/translate <nick> disable
/translate <nick> lang=es
```

**Caso de uso:**
- Usuario A habla español
- Usuario B habla inglés
- Sistema traduce automáticamente (opcional)
- Ambos ven respuestas en su idioma

**Propiedades:**
- Opt-in, no obligatorio
- Usa traducción local (no terceros)
- Idiomas detectados automáticamente

---

### `PASTE` - Compartir Código

```
/paste <idioma> [duración]
/paste python expire=1h
```

**Resultado:**
```
→ Sistema abre sesión de paste
→ Usuario pega código
→ Sistema genera URL de corta duración
→ Mensaje: "Código compartido: https://ircpilot.local/paste/abc123 (expira en 1h)"
```

**Propiedades:**
- Syntax highlighting
- Duración configurable (5 min a 1 semana)
- Privado o público
- Descargable

---

### `STATS` - Estadísticas Personales

```
/stats [nick]
/stats me
/stats #canal
```

**Salida:**
```
ESTADÍSTICAS DE alice_fp (última semana):
  Mensajes: 342
  Canales: 8
  Promedio msg/día: 49
  Hora pico: 22:00-23:00
  Palabra favorita: "interesante"
  
Reputación: +28 (👍 23 usuarios)
Trivia ganada: 5 juegos
Membresía: 2 años
```

---

### `FEEDBACK` - Reporte de Bugs/Sugerencias

```
/feedback bug <descripción>
/feedback suggest <idea>
/feedback list [status]
```

**Caso de uso:**
- Usuario reporta bug
- Sistema lo archiva
- Comunidad puede votar si es válido
- Desarrolladores ven prioridades reales

---

## 🎨 Categoría 6: Personalización

### `THEME` - Temas Visuales (Cliente)

```
/theme list
/theme set <nombre>
/theme create <nombre> [base_colors]
```

**Temas incluidos:**
- `light` - Claro y limpio
- `dark` - Oscuro y relajante
- `retro` - Estilo años 90
- `neon` - Cyberpunk
- `minimal` - Ultra simple
- `color_blind` - Accesible para daltonismo

---

### `PROFILE` - Perfil Público Personalizable

```
/profile set bio <texto>
/profile set avatar <url>
/profile set color <hex>
/profile set interests <lista>
/profile public <yes|no>
```

**Perfil visible:**
```
alice_fp 🎮
  "Gamer, Python dev, cat lover"
  Intereses: Gaming, Programming, Anime
  Reputation: ⭐⭐⭐⭐ (+47)
  Joined: 2 years ago
```

---

## 🌐 Categoría 7: Integración y Extensibilidad

### `BOT` - Sistema de Bots Comunitarios

```
/bot create <nombre> <tipo>
/bot edit <bot_id> <config>
/bot delete <bot_id>
/bot market (explorar bots de la comunidad)
```

**Tipos de bots:**
- `Weather` - Reporta clima
- `Translate` - Traduce mensajes
- `News` - Noticias automáticas
- `Music` - Comparte canciones
- `Reminder` - Recordatorios
- `Custom` - Script personalizado (Lua/WASM)

**Ejemplo:**
```
Usuario A crea bot de clima:
  /bot create weather_bot weather
  /bot edit weather_bot location="Buenos Aires"
  
En canal:
  Usuario B: /bot weather_bot
  → Bot: "Buenos Aires: 22°C, Parcialmente nublado, Viento 10km/h"
```

---

### `PLUGIN` - Extensión de Protocolo

```
/plugin list
/plugin install <plugin_name>
/plugin enable #canal
/plugin disable #canal
```

**Plugins comunitarios:**
- `Moderation+` - Herramientas extra de moderación
- `Games+` - Más juegos
- `Stats+` - Estadísticas avanzadas
- `Theme+` - Más temas

---

## 📊 Categoría 8: Información y Metadatos

### `WHOIS` - Información de Usuario

```
/whois <nick|fingerprint>
```

**Salida:**
```
alice_fp (alice, alice_gaming)
  Status: Online (2 hours)
  Joined: 2 years ago
  Reputation: ⭐⭐⭐⭐ (+47)
  
  Últimas actividades:
    - #general: "Hola gente!"
    - #gaming: Ganó trivia (5 puntos)
    - #tech: Bookmarked mensaje sobre Rust
  
  Intereses: Gaming, Python, Anime
  Website: [Profile]
```

---

### `ACTIVITY` - Mostrar Actividad Global

```
/activity [tipo] [periodo]
/activity messages last_hour
/activity games today
/activity new_users this_week
```

**Salida:**
```
ACTIVIDAD ÚLTIMAS 24 HORAS:
  Mensajes: 2,341
  Usuarios activos: 87
  Canales con actividad: 23
  Juegos iniciados: 15
  Nuevos usuarios: 3
  
Top canal: #general (523 mensajes)
```

---

### `SEARCH` - Búsqueda Global

```
/search <término> [in:canal] [from:nick] [type:message|trivia|wiki]
/search "python tutorial" in:#tech from:alice
/search "¿Cómo aprender?" type:wiki
```

---

## 🎁 Categoría 9: Rewards y Incentivos

### `ACHIEVEMENT` - Logros y Medallas

```
/achievement list
/achievement progress <achievement_id>
```

**Logros disponibles:**
- `First Chat` - Primer mensaje
- `Chatty` - 100 mensajes en 1 día
- `Trivia Master` - Ganar 10 trivias
- `Social Butterfly` - Unirse a 10 canales
- `Helper` - Dar 5 reps positivos
- `Controversial` - 10 warns diferentes
- `Speedrunner` - Ganar 3 juegos en 1 hora
- `Historian` - Crear wiki page
- `Civic Duty` - Votar en 5 encuestas

**Salida:**
```
LOGROS DESBLOQUEADOS:
  🏆 First Chat (1 Oct 2024)
  🎮 Trivia Master (15 Sep 2024)
  🦋 Social Butterfly (7 Aug 2024)
```

---

### `REWARD` - Sistema de Puntos Virtuales

```
/reward balance
/reward shop
/reward buy <item_id>
```

**Items de tienda:**
- `Custom Emoji` - Crear emoji personalizado (100 pts)
- `Channel Theme` - Cambiar tema del canal (500 pts)
- `Bot Slot` - Crear bot adicional (250 pts)
- `Private Wiki` - Wiki privada del usuario (1000 pts)
- `Profile Banner` - Banner personalizado (200 pts)

**Ganancias de puntos:**
- Por mensaje: 1 pt
- Por trivia ganada: 10 pts
- Por rep recibida: 5 pts
- Por logro desbloqueado: 25 pts
- Por evento asistido: 10 pts

---

## 📋 Categoría 10: Gestión de Contenido

### `ARCHIVE` - Archivado de Canales

```
/archive create <nombre>
/archive upload <canal>
/archive download <archive_id>
/archive list
```

**Caso de uso:**
- Comunidad guarda historial de canal
- Genera PDF/HTML descargable
- Recurso educativo permanente

---

### `EXPORT` - Exportar Datos Personales

```
/export all
/export messages [canal] [período]
/export profile
/export stats
```

**GDPR compliance:**
- Usuario puede descargar todos sus datos
- Formato estándar (JSON/CSV)
- Incluye mensajes, perfil, stats, medallas

---

## 🎯 Tabla de Priorización

| Categoría | Impacto | Complejidad | Fase |
|-----------|---------|-------------|------|
| **Juegos** | Alto | Media | 2-3 |
| **Comunicación Enriquecida** | Alto | Baja | 1-2 |
| **Conocimiento** | Muy Alto | Alta | 2-3 |
| **Comunidad** | Alto | Media | 2 |
| **Utilidad** | Alto | Baja | 1-2 |
| **Personalización** | Medio | Baja | 2 |
| **Integración** | Muy Alto | Muy Alta | 3-4 |
| **Información** | Medio | Baja | 1-2 |
| **Rewards** | Medio | Media | 3 |
| **Gestión de Contenido** | Medio | Baja | 3 |

---

## 🔄 Flujos de Ejemplo Integrando Nuevos Comandos

### Flujo 1: Noche de Trivia

```
20:00 - Usuario A organiza evento:
  /event create "Trivia Night 2026-10-20" "Trivia en directo"
  → Sistema notifica a 50 usuarios suscritos
  
20:25 - Usuarios se unen:
  /event join 42
  → Se crea canal temporal #event_42
  
20:30 - Comienza:
  /game start trivia difficulty=hard category=movies
  → 15 usuarios compiten
  → Alice gana (7/10 correctas)
  
20:45 - Resultados:
  /leaderboard games today
    1. alice (+50 pts, +10 reward pts)
    2. bob (+40 pts, +8 reward pts)
  
21:00 - Celebración:
  /achievement progress "Trivia Master"
  /emote use party party party
  /rep give alice "organizaste evento genial"
```

---

### Flujo 2: Proyecto Colaborativo

```
Lunes - Idea:
  /poll create "¿Hacemos guía de Python?" "Sí" "No" "Quizá"
  → Resultados: 23 votos "Sí"
  
Martes - Creación:
  /wiki create "Guía Completa Python"
  /wiki edit page_id "[Contenido inicial]"
  
Semana - Colaboración:
  Usuario A: /wiki edit page_id "## Variables"
  Usuario B: /wiki edit page_id "## Funciones"
  Usuario C: /wiki edit page_id "## Clases"
  
Viernes - Finalización:
  /wiki link page_id (aparece en descripción del canal)
  Creador A recibe: +25 achievement pts, +10 reputation
  
Usuarios descubren:
  /search "python tutorial"
  → Encuentra la wiki
  /bookmark add wiki_page "python_guide"
```

---

## 📝 Notas de Evolución

- **MVP (Fase 0.1)**: Core + IGNORE + IDENTIFY
- **Fase 1**: WARN + MODERATE + Utilidad básica (STATS, WHOIS, BOOKMARK)
- **Fase 2**: Juegos (TRIVIA, GAMES), Conocimiento (WIKI, POLL), Comunidad (REPUTATION, EVENT)
- **Fase 3**: Personalización (PROFILE, THEME), Rewards (ACHIEVEMENT, REWARD)
- **Fase 4+**: Integración (BOT, PLUGIN), Análisis avanzado

**La clave**: Cada comando debe aumentar utilidad sin sacrificar privacidad o justicia.

