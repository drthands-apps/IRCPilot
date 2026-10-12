# IRCPilot - Especificación de Comandos y Semántica

> **Documento de Diseño**: Define los comandos del sistema, su semántica y diferencias con IRC clásico. Es la base para el motor de interpretación.

---

## 🎯 Principio de Diseño

**No copiamos IRC. Lo rediseñamos desde cero respetando su usabilidad pero cumpliendo nuestros objetivos de privacidad, seguridad y protección del usuario anónimo.**

Los comandos son la "interfaz de derechos": cada comando refleja un derecho del usuario y una regla de moderación.

### Filosofía de Responsabilidad y Moderación

- **El usuario es responsable de su propia defensa**: El sistema proporciona herramientas (IGNORE, LANG_FILTER, VISIBILITY) pero no "protege" paternalistamente.
- **El sistema es transparente, no paternalista**: El servidor no notifica al ignorado que ha sido ignorado. Cada usuario gestiona su propio filtro.
- **Moderación es delegada al usuario cuando es posible**: Ignore permanente es una forma de "moderación individual". El servidor facilita, no ordena.
- **Gestión activa de datos**: Los fingerprints inactivos acumulan ignore-lists innecesarias. Se requiere confirmación periódica, no borrado silencioso.

**PERO: Moderación activa es responsabilidad del servidor en casos graves.**

La ética de cada canal define qué es "tolerable". El servidor proporciona herramientas pero los **moderadores garantizan las reglas del canal**, priorizando soluciones educativas antes que punitivas, pero responsabilizando al atacante de su actitud.

---

## 📋 Categorías de Comandos

### Categoría A: Gestión de Sesión y Identidad
- `NICK` - Cambiar nickname
- `USER` - Autenticarse (redefinido)
- `QUIT` - Desconectar
- `IDENTIFY` - Sistema de identidad no-IP

### Categoría B: Navegación de Canales
- `JOIN` - Entrar a canal
- `PART` - Salir de canal
- `LIST` - Listar canales
- `TOPIC` - Ver/cambiar tema

### Categoría C: Comunicación
- `PRIVMSG` - Mensaje privado
- `PUBMSG` - Mensaje en canal
- `NOTICE` - Notificación (redefinido)

### Categoría D: Control de Privacidad y Filtrado
- `IGNORE` (redefinido con variantes)
- `LANG_FILTER` (nuevo)
- `VISIBILITY` (nuevo)

### Categoría E: Moderación y Justicia
- `REPORT` - Reportar abuso (nuevo)
- `MODERATE` - Acciones de moderador
- `APPEAL` - Apelar sanción (nuevo)
- `AUDIT_LOG` - Ver auditoría (nuevo)
- `WARN` - Advertencia educativa (nuevo)

### Categoría F: Reglas y Gobernanza
- `RULES` - Ver/establecer reglas de canal
- `VOTE` - Sistema de votación (futuro)

### Categoría G: Gestión de Datos y Privacidad
- `IGNORE LIST` - Ver tus ignores activos
- `IGNORE MAINTENANCE` - Revisar ignores de fingerprints inactivos (nuevo)
- `PRIVACY STATUS` - Ver tu privacidad actual (nuevo)

---

## 🔐 Comandos de Privacidad y Filtrado (CRÍTICOS)

### `IGNORE` - Sistema de Ignorado Redefinido

IRC clásico tiene un `IGNORE` simple y unidireccional. Aquí definimos **4 tipos** según necesidad:

#### Tipo 1: `IGNORE SESSION`
```
/ignore <nick> session
```
- **Alcance**: Solo esta conexión/sesión
- **Duración**: Hasta desconectar o `UNIGNORE`
- **Efecto**: No ves mensajes de `<nick>`
- **Bilateral**: NO (solo tú ignoras)
- **Visibilidad**: `<nick>` NO sabe que lo ignoras (ni notificación, ni silencio: el usuario es responsable)

**Caso de uso**: Usuario molesto en una charla puntual.

**Nota**: El usuario ignorado puede seguir spammeando / floodfood el servidor, pero no te molestará a ti. Es responsabilidad del usuario ignorado no hacerlo, y del servidor detectar patrones de flood via otros métodos (rate-limiting).

---

#### Tipo 2: `IGNORE PERMANENT`
```
/ignore <nick|fingerprint> permanent
```
- **Alcance**: Todos tus nicknames en IRCPilot (vinculado a identidad)
- **Duración**: Indefinido (hasta `UNIGNORE`)
- **Efecto**: No ves mensajes de `<fingerprint>` en ningún contexto
- **Bilateral**: NO
- **Visibilidad**: El usuario ignorado NO sabe que lo ignoras (ni recibe notificación)

**Caso de uso**: Usuario permanentemente molesto que querés bloquear.

**Nota importante**: El usuario ignorado puede seguir siendo activo (spam, flood) en canales y no afectarte a ti, pero TAMBIÉN molestará a otros usuarios. Esto es correcto: no es tu responsabilidad regular su comportamiento global, solo tu propia experiencia.

**Mantenimiento de ignores**: Ver sección "Gestión de Inactividad en Ignores".

---

#### Tipo 3: `IGNORE BILATERAL` (MUTUAL BLOCK)
```
/ignore <nick|fingerprint> bilateral
```
- **Alcance**: Bidireccional completo
- **Duración**: Indefinido
- **Efecto**: 
  - Ni tú ves mensajes de `<fingerprint>`
  - Ni `<fingerprint>` ve mensajes tuyos
  - Si ambos estáis en un canal, ni vuestros nicks aparecen en mensajes del otro
  - En privado: PRIVMSG rechazado con "Usuario bloqueado"
- **Bilateral**: SÍ (ambos quedan separados)
- **Visibilidad**: Ambos saben que están mutuamente ignorados (es una acción consensual de dos partes)

**Caso de uso**: Dos usuarios que no quieren interactuar en absoluto. Separación consensual.

**Implementación técnica**: Requiere que el servidor filtre mensajes bidireccionales, oculte presencia y no incluya nicks en contextos compartidos.

---

#### Tipo 4: `IGNORE APPEAL` (Apelación contra Baneo/Silencio)
```
/ignore <nick> appeal
```
Este no es un ignore normal. Es un "marcador" que indica: "He sido sancionado por este moderador y quiero apelar".

Ver sección de **Moderación y Apelación**.

---

### `LANG_FILTER` - Filtro de Lenguaje por Nivel

**Nuevo comando que permite filtrar contenido por edad/sensibilidad.**

```
/lang_filter <nivel>
```

Niveles:
- `G` - General (sin contenido adulto)
- `PG` - Parental Guidance (lenguaje moderado)
- `R` - Restringido (lenguaje adulto)
- `OFF` - Sin filtro

**Comportamiento:**
- Usuario marca su nivel aceptable
- Cuando recibe mensaje, servidor verifica tag de contenido del remitente
- Si el mensaje supera nivel del usuario, se marca como `[CONTENIDO ADULTO - Click para ver]` o similar
- Usuario puede expandir manualmente o ignorar

**Implementación:**
- Remitente declara nivel de contenido: `/content_level <nivel>`
- Sistema verifica consistencia (antispam)
- Servidor filtra según preferencia receptor

**Caso de uso:**
- Usuario joven quiere chat limpio
- Usuario adulto quiere chat libre
- Ambos pueden coexistir en mismo canal sin conflicto

---

### `VISIBILITY` - Control de Presencia

```
/visibility <modo>
```

Modos:
- `PUBLIC` - Visible en listados de usuarios (default)
- `STEALTH` - No apareces en `/WHO`, solo en canales si estás presente
- `GHOST` - No apareces en nada ni nadie sabe que estás conectado (excepto a quien envíes privmsg)

**Caso de uso:**
- Usuarios que quieren lurk sin ser vistos
- Investigadores de abuso
- Usuarios que simplemente prefieren privacidad

---

## 👥 Sistema de Identidad (NO-IP)

### Problema
- IP es fácil de falsificar (VPN, proxy)
- Email es invasivo y rastreable
- Nick se puede cambiar

### Solución: FINGERPRINT de Sesión + Desafío Criptográfico

```
/identify <passphrase>
```

**Proceso:**
1. Usuario define una passphrase (privada, no transmitida al servidor)
2. Servidor genera SALT criptográfico aleatorio
3. Cliente computa: `FINGERPRINT = HMAC_SHA256(passphrase + SALT)`
4. Cliente envía `FINGERPRINT` (no la passphrase)
5. Servidor lo vincula a la identidad y lo almacena

**Propiedades:**
- No es IP (no cambia si cambias red)
- Es persistente (vinculado a passphrase + SALT)
- No es email (privado)
- Difícil falsificar: cambiar identidad requiere recordar/regenerar passphrase
- Cambiar identidad es incómodo pero posible (no imposible, respeta legalidad)

**Ventajas:**
- Cada passphrase = identidad única
- Baneos afectan identidad, no nick/IP
- Usuario puede tener múltiples identidades (cada una con su passphrase)
- Sistema completamente descentralizable si lo deseas

**Notas de seguridad:**
- SALT es público (se guarda en cliente)
- FINGERPRINT nunca es la passphrase (derivación segura)
- Servidor nunca ve passphrase
- Cambiar passphrase = nueva identidad

---

## 🛡️ Protección Contra Flood/Spam (Responsabilidad del Servidor)

Mientras que la protección individual es responsabilidad del usuario (IGNORE), el servidor TIENE la responsabilidad de evitar ataques al sistema mismo.

### Rate Limiting por Fingerprint
```
- Max 5 mensajes por segundo
- Max 100 mensajes por minuto
- Max 1000 mensajes por hora
- Exceso: Mute automático por 5 minutos
```

### Detección de Flood
```
- >20 mensajes idénticos en 30 segundos = Ban temporal
- >50 menciones de un usuario en 1 minuto = Ban temporal
- >100 canales joinados en 1 minuto = Ban temporal (bot detection)
```

### Comportamiento
- Infracciones se registran en auditoría
- Usuario recibe NOTICE explicando limite
- No se notifica públicamente (no alertamos al atacante)
- Límites se resetean después de período sin infracciones

---

## 🚨 Moderación, Escalada y Justicia

### Modelo de Escalada de Moderación

La moderación NO es punitiva por defecto. Es **educativa y progresiva**. Solo se vuelve punitiva si el atacante muestra actitud de resistencia o reincidencia.

#### Nivel 1: Solución Individual (Usuario → Usuario)
**Herramientas disponibles para el usuario afectado:**
- `IGNORE SESSION` - Ignorar temporalmente
- `IGNORE PERMANENT` - Bloquearlo permanentemente
- `IGNORE BILATERAL` - Separación mutua

**Objetivo**: El usuario afectado se auto-protege sin intervención del servidor.

---

#### Nivel 2: Advertencia Educativa (Moderador → Usuario)
**Comando**:
```
/warn <nick|fingerprint> <razón> [extra_info]
```

**Propiedades:**
- No es punitivo, es informativo
- Usuario recibe NOTICE: "Advertencia: tu comportamiento de [razón] viola regla #X del canal"
- Se guarda en auditoría (conteo de advertencias)
- Usuario NO es silenciado o baneado

**Razones típicas:**
- Spam repetitivo (primer aviso)
- Lenguaje inapropiado según reglas del canal
- Acoso a otro usuario (primer aviso)
- Flood (primer aviso)

**Escenario:**
```
Usuario A insulta a Usuario B
  → Moderador ejecuta: /warn A "insultos contra otros usuarios"
  → A recibe: "ADVERTENCIA: Los insultos contra otros usuarios violan regla #3. Recordamos que este es un espacio de respeto."
  → Se registra en auditoría: "A warned for insults, 1st warning, timestamp"
```

**Decisión del atacante:**
- ✓ Cambia de actitud → Sigue adelante (warning limpio después de 30 días sin reincidencia)
- ✗ Ignora advertencia → Escala a Nivel 3

---

#### Nivel 3: Corrección Temporal (Moderador → Usuario)
**Comando**:
```
/moderate <nick|fingerprint> mute <segundos> <razón>
```

**Propiedades:**
- Usuario es silenciado por X segundos
- Recibe NOTICE explicando la acción y la razón
- Se registra en auditoría
- Usuario PUEDE APELAR inmediatamente

**Cuándo aplicar:**
- Reincidencia después de advertencia
- Comportamiento grave pero no ban-worthy (ej: mensajes ofensivos pero no hate speech)
- Flood continuado

**Escenario:**
```
Usuario A fue advertido hace 2 horas por insultos
  → A vuelve a insultar a C
  → Moderador ejecuta: /moderate A mute 300 "reincidencia en insultos post-advertencia"
  → A recibe: "MODERACIÓN: Silenciado por 5 minutos. Razón: reincidencia en insultos post-advertencia. Puedes apelar con /appeal."
  → A no puede enviar mensajes por 300 segundos
  → Después del mute, A puede intentar de nuevo (2ª oportunidad)
```

---

#### Nivel 4: Baneo Temporal (Moderador → Usuario)
**Comando**:
```
/moderate <fingerprint> ban <segundos> <razón>
```

**Propiedades:**
- Usuario es expulsado del canal por X segundos/horas
- No puede rejoin hasta que expire
- Recibe NOTICE con auditoría de por qué
- Usuario PUEDE APELAR

**Cuándo aplicar:**
- Múltiples reincidencias (3+ avisos o mutes)
- Comportamiento grave (hate speech, acoso persistente, threats)
- Reincidencia después de mute en misma sesión

**Duración típica:**
- Primera vez: 24 horas
- Segunda vez: 7 días
- Tercera vez: 30 días (o permanente si la comunidad vota)

**Escenario:**
```
Usuario A: Advertido (insultos)
         → Silenciado 5 min (reincidencia)
         → Silenciado 5 min again (reincidencia)
         → Baneado 24 horas (3ª reincidencia en sesión)
  
Usuario A recibe: "BAN: Expulsado por 24 horas. Razón: reincidencia persistente en comportamiento abusivo (3 sanciones en sesión). Puedes apelar con /appeal."
```

---

#### Nivel 5: Baneo Permanente (Comunidad Votación)
**Comando**:
```
/moderate <fingerprint> permaban <razón> [vote_required]
```

**Propiedades:**
- Baneo indefinido
- Requiere votación de moderadores (no decisión unilateral)
- Usuario puede apelar en cualquier momento
- Se documenta exhaustivamente

**Cuándo aplicar:**
- Comportamiento tóxico sin remedio (reincidencia extrema)
- Acoso sexual, amenazas de violencia
- Spam/bot coordinado
- Violación grave de leyes (ej: incitación al odio, doxxing)

**Votación:**
- Quórum: 3+ moderadores
- Mayoría: 2/3+ para sostener
- Usuario ve evidencia y puede defender

---

### `WARN` - Advertencia Educativa (Nuevo)

```
/warn <nick|fingerprint> <razón> [nota_privada]
```

**Registro:**
- Se guarda en auditoría de usuario
- Conteo: "Usuario X tiene 2 warnings en 30 días"
- Visible para otros moderadores

**Expiración:**
- Warning se considera "resuelto" después de 30 días sin reincidencia
- Usuario puede ver sus warnings activos: `/audit_log my_warnings`

**Notas para moderadores:**
```
/warn alice "spam en #general" "3er mensaje idéntico en 2 min, le dimos 1 chance"
```

**Nota privada** solo la ven moderadores, no el usuario.

---

### Sistema de Moderación Automática (IA/Heurística)

```
/report <nick> <razón>
```

**Proceso:**
1. Usuario reporta conversación o nick
2. Sistema analiza automáticamente:
   - Patrones de spam (mensajes repetitivos)
   - Hate speech (palabras clave según reglas del canal)
   - Acoso (menciones agresivas repetidas)
   - Flood (demasiados mensajes en poco tiempo)
3. Si score > umbral, sugiere acción a moderadores
4. Moderadores revisan y deciden (pueden ignorar, advertir, mute, ban)

**Diseño de privacidad:**
- Análisis con hash de palabras, no contenido plano
- No se almacenan transcripciones
- Solo metadatos (quién, cuándo, score)

**Salida del sistema:**
```
ALERT: Possible abuse pattern detected
  - User: eve_fp
  - Reason: Flood
  - Score: 8.5/10
  - Action: Suggested MUTE 300s
  - Evidence: 25 messages in 3 minutes in #general

Moderators: /moderate eve_fp mute 300 "auto-detected flood"
           or /warn eve_fp "spam pattern" "bot-detected, 1st warning"
           or /ignore (do nothing)
```

---

### Sistema de Apelación (Justicia Descentralizada)

Si un usuario es baneado, silenciado o advertido, puede apelar:

```
/appeal <mod_nick> <razón>
```

**Proceso:**
1. Usuario apela acción de moderador
2. Sistema crea ticket de apelación
3. Quórum de otros moderadores revisa (o comisión de ética)
4. Se requiere mayoría para sostener sanción
5. **Transparencia**: Usuario puede ver razonamiento y audiencia

**Garantías:**
- No es el mismo moderador quien decide la apelación
- Registro auditable
- Usuario ve evidencia en su contra
- Puede presentar defensa

**Ejemplo:**
```
/appeal moderator_bob "fue en broma, no sabía que violaba la regla"
  → Sistema notifica a 3 otros moderadores
  → Bob ve apelación pero NO vota (conflicto de interés)
  → Carol, Dave, Eve revisan evidencia
  → Votación: Carol (mantener ban), Dave (reduce a mute), Eve (remover)
  → Resultado: 2-1 para mantener
  → Bob recibe: "Appeal denied. Moderators voted 2-1 to maintain ban. You can appeal again in 7 days."
```

---

### `AUDIT_LOG` - Ver Auditoría

```
AUDIT_LOG [#canal|nick] [last <N> lines]
AUDIT_LOG search <pattern>
```

**Tipos de eventos auditados:**
- Advertencias (WARN)
- Silenciamientos (MUTE)
- Baneo/Desbaneo
- Cambios de reglas
- Reports
- Apelaciones
- Acciones de moderador
- Cambios de IDENTIFY
- Cambios de configuración

**Restricciones:**
- Usuario normal: solo ve sus propios eventos
- Moderador: ve eventos de su canal + reports
- Admin: ve todo
- Timestamps encriptados con privacidad

**Salida típica:**
```
AUDIT_LOG my_warnings
  2026-10-12 14:30 - WARN: spam in #general (moderator: alice)
  2026-09-15 09:45 - WARN: offensive language (moderator: bob)

AUDIT_LOG #general warnings
  2026-10-12 15:22 - eve_fp WARN: flood (1st warning)
  2026-10-12 15:21 - charlie_fp WARN: insultos (1st warning)
```

---

## 📋 Comandos Específicos Redefinidos

### `USER` (Redefinido para Anonimato)

IRC clásico:
```
USER <username> <hostname> <servername> <realname>
```

IRCPilot:
```
USER <anonymous> <hidden> <ircpilot> <anonymous>
```

**Cambio**: No pedimos datos reales. Usuario siempre es "anonymous", la identidad es el FINGERPRINT.

---

### `PRIVMSG` (Con tags de privacidad)

```
PRIVMSG <target> <message>
```

Tags implícitos:
- `+no_history` - Mensaje no se guarda (solo en caché temporal)
- `+ephemeral <seconds>` - Mensaje desaparece en X segundos
- `+no_forward` - Destinatario no puede forwarding
- `+content_level <G|PG|R>` - Nivel de contenido

**Ejemplo:**
```
PRIVMSG #general :Hola, esto es un mensaje normal
@no_history;content_level=R PRIVMSG #general :Esto no se guarda
```

---

### `NOTICE` (Redefinido: Notificaciones del Sistema)

```
NOTICE <nick> <mensaje>
```

Se usa para:
- Auditoría: "Tu mensaje fue reportado"
- Moderación: "Has sido silenciado por 10 minutos"
- Sistema: "Se actualizaron reglas del canal"
- Advertencias: "ADVERTENCIA: ..."

**Importante**: NO se usa para comunicación usuario-usuario (eso es PRIVMSG).

---

### `QUIT` (Redefinido: Con Limpieza)

```
QUIT :<mensaje_de_salida>
```

Cuando un usuario se desconecta:
1. Se borra su presencia de todos los canales
2. Sus mensajes privados se marcan como "enviados pero no entregados"
3. Su identidad queda "activa" (para baneo persistente si aplica)
4. Se ejecuta hook de limpieza (auditoría)

---

## 🎯 Comandos Nuevos Específicos de IRCPilot

### `RULES` - Sistema de Reglas de Canal

```
RULES [#canal]
RULES #canal set <rule_id> <descripción>
RULES #canal delete <rule_id>
RULES #canal vote <rule_id> <yes|no>
```

**Propiedades:**
- Reglas son **inmutables una vez fijadas** (solo pueden ser reemplazadas con votación)
- Se guardan en cadena de auditoría
- Cada modificación queda registrada
- Usuarios pueden votar cambios de reglas
- Moderadores pueden invocar reglas en decisiones de moderación

**Ejemplo de reglas de canal:**
```
#gaming:
  1. No spam links (exception: official game guides)
  2. No hate speech (includes slurs, misogyny, racism)
  3. Spoilers must be tagged: [SPOILER Game Name]
  4. Be respectful to everyone
  
#adult:
  1. Contenido adulto explícito permitido pero marcar con @content_level=R
  2. No non-consensual content
  3. Be respectful even in adult context
```

---

### `MODERATE` - Acciones de Moderador

```
MODERATE <nick|fingerprint> <acción> <duración> [razón]
```

Acciones:
- `WARN <razón>` - Advertencia educativa (ver sección WARN)
- `MUTE <segundos>` - Silenciar por X segundos
- `KICK` - Expulsar del canal (no baneo permanente)
- `BAN <segundos>` - Baneo temporal a identidad
- `PERMABAN` - Baneo permanente (requiere votación)
- `UNBAN <identidad>` - Desbanear

**Importante:**
- Baneo es por IDENTIDAD (fingerprint), no por nick/IP
- Cada acción queda auditada
- Moderador debe dar razón (en auditoría)
- Usuario puede apelar cada acción

**Ejemplos:**
```
/moderate alice warn "spam en canal"
/moderate bob mute 300 "reincidencia post-advertencia"
/moderate charlie ban 86400 "hate speech grave"
/moderate dave permaban "acoso sexual persistente" vote_required
```

---

### `IGNORE LIST` - Ver tus ignores activos

```
/ignore list
```

**Salida:**
```
SESSION IGNORES:
  - alice (expires in 1 hour)
  - bob (expires at disconnect)

PERMANENT IGNORES:
  - charlie_fp (active 2 years)
  - dave_fp (active 6 months)

BILATERAL BLOCKS:
  - eve_fp (mutual, active 1 year)
```

---

### `IGNORE MAINTENANCE` - Gestión de Inactividad (NUEVO)

**Problema**: A medida que pasan meses o años, algunos fingerprints ignorados se vuelven inactivos. Acumular ignores de usuarios muertos es ineficiente.

```
/ignore maintenance
```

**Proceso:**
1. Sistema identifica ignores cuyo fingerprint lleva >6 meses inactivo
2. Pregunta al usuario: "¿Todavía querés ignorar a `charlie_fp` que lleva 8 meses inactivo? (SI/NO/DELETE)"
3. Opciones:
   - `SI` - Mantener ignore (puede que vuelvan algún día)
   - `NO` - Desactivar temporalmente pero guardar en historial
   - `DELETE` - Borrar completamente

**Beneficio**: 
- Limpia datos obsoletos
- Respeta decisión del usuario (no borra silenciosamente)
- Permite "regresar" usuarios si vuelven

**Timing**: 
- Se sugiere automáticamente cada 6 meses
- Usuario puede ejecutarlo manualmente en cualquier momento

---

### `PRIVACY STATUS` - Ver tu configuración de privacidad (NUEVO)

```
/privacy status
```

**Salida:**
```
IDENTITY:
  - Fingerprint: a3b2c1d0e9f8g7h6
  - Active nicknames: 3
  - Account age: 2 years

FILTERS:
  - Language filter: R (adult content)
  - Visibility: PUBLIC
  
PROTECTIONS:
  - Session ignores: 2
  - Permanent ignores: 5
  - Bilateral blocks: 1
  - Rate-limit status: NORMAL (234/1000 messages/hour)

MODERATION:
  - Active mutes: 0
  - Active bans: 0
  - Active warnings: 1 (spam warning from 3 days ago)
  - Appeals pending: 0
  - Last report received: 2 weeks ago
```

---

## 📊 Tabla Comparativa: IRC Clásico vs IRCPilot

| Aspecto | IRC Clásico | IRCPilot |
|---------|-------------|----------|
| **IGNORE** | Unidireccional, básico | 4 tipos (session, permanent, bilateral, appeal) |
| **Identidad** | Nick + IP | Fingerprint criptográfico |
| **Privacidad Contenido** | Servidor ve todo | Mensajes E2E encriptados |
| **Moderación** | Manual, opaca, punitiva | Educativa, escalada, transparente |
| **Advertencias** | No existe | Sí, con educación antes de castigo |
| **Reglas** | Informales | Formales, inmutables, auditadas, flexibles por canal |
| **Auditoría** | Ninguna | Completa y accesible |
| **Apelación** | No existe | Sí, con votación y defensa |
| **Visibilidad Presencia** | Siempre público | Configurable (PUBLIC/STEALTH/GHOST) |
| **Baneos** | Por IP (falsificable) | Por identidad (difícil falsificar) |
| **Flood Protection** | Basic rate-limit | Sofisticado, por fingerprint |
| **Inactivity Management** | N/A | Check-in interactivo en ignores |
| **Ética de Canal** | Centralizada | Localizada en reglas del canal |

---

## 🔄 Flujos de Ejemplo

### Flujo 1: Usuario Joven Ingresa a Canal Adulto

```
Usuario A (edad 14) se une a #general
  → Servidor nota que LANG_FILTER de A es 'G'
  → Usuario B envía mensaje con content_level='R'
  → Servidor muestra a A: "[CONTENIDO ADULTO] Usuario B dice algo. [Click para expandir]"
  → A no puede ver contenido adulto a menos que lo expanda manualmente
  → Ambos pueden convivir sin conflicto
```

---

### Flujo 2: Escalada de Moderación (Abuso con Actitud)

```
Usuario A empieza a insultar a Usuario B en #general:

INSTANCIA 1: Primer insulto
  → Moderador alice ejecuta: /warn A "insultos contra otro usuario - regla #3"
  → A recibe NOTICE: "ADVERTENCIA: Los insultos contra otros usuarios violan regla #3. Este es un espacio de respeto."
  → Se registra: warn count = 1

INSTANCIA 2: 30 minutos después, A vuelve a insultar a B
  → Moderador bob ejecuta: /moderate A mute 300 "reincidencia post-advertencia"
  → A recibe NOTICE: "MODERACIÓN: Silenciado por 5 minutos por reincidencia. Razón: continúa insultando post-advertencia."
  → A no puede hablar por 300 segundos
  → Se registra: warn count = 1 + mute 1

INSTANCIA 3: Después del mute, A sale y vuelve a entrar, reincide
  → Moderador carol ejecuta: /moderate A ban 86400 "3ª reincidencia en insultos en 1 hora"
  → A recibe NOTICE: "BAN: Expulsado del canal por 24 horas. Razón: comportamiento abusivo persistente después de advertencia + mute."
  → A puede apelar: /appeal carol "no sabía que era tan serio"
  
INSTANCIA 4: Apelación
  → Panel de moderadores (bob, dave, eve) revisa
  → Carol no vota (conflicto de interés)
  → Bob: MANTENER (atacante reincide deliberadamente)
  → Dave: REDUCIR A MUTE (primera vez merecía oportunidad)
  → Eve: REMOVER (fue en broma)
  → Resultado: 1-2 para remover ban
  → A es desbaneado pero con ADVERTENCIA permanente en expediente
```

**Clave**: La moderación fue **educativa** (warn), escaló solo porque **la actitud del atacante fue de resistencia** (ignorar advertencias), y la apelación permitió **revisar si fue proporcional**.

---

### Flujo 3: Ignorar Permanente (Solución Individual)

```
Usuario A es molesto pero NO violador de reglas
  → Usuario B ejecuta: /ignore A permanent
  → B no ve mensajes de A
  → A puede seguir escribiendo (es libre de hacerlo)
  → Si A hace spam, el servidor lo detecta y lo mute (protección global)
  → B no se afecta (ya lo ignora)
  → Otros usuarios que NO ignoran a A, si ven el mute del servidor
```

---

### Flujo 4: Mutual Block (Separación Completa)

```
Usuario A y B tuvieron discusión tóxica
  → A ejecuta: /ignore B bilateral
  → Sistema obtiene confirmación de ambos (o confirmación unilateral + consentimiento implícito)
  → Resultado:
    - A no ve mensajes de B
    - B no ve mensajes de A
    - Si ambos están en #general:
      - A ve: "Usuario anónimo dijo algo" (nick de B oculto)
      - B ve: "Usuario anónimo dijo algo" (nick de A oculto)
    - En privado: PRIVMSG rechazado con "Usuario bloqueado"
```

---

### Flujo 5: Mantenimiento de Ignores

```
Usuario A verificó ignore list hace 2 años
  → Charlie_fp fue ignorado permanente pero lleva 9 meses sin conectar
  → Sistema sugiere: "Charlie_fp ha estado inactivo 9 meses. ¿Mantener ignore? (SI/NO/DELETE)"
  → A elige SI (por si vuelve algún día)
  → charlie_fp se mantiene en lista "Inactive Ignores" (bajo storage)
  → Si charlie_fp vuelve: Sistema reactivará ignore automáticamente
```

---

### Flujo 6: Reporte de Abuso y Acción Automática

```
Usuario B reporta a Usuario A por spam:
  /report A "spam repetitivo en #general"
  
Sistema analiza:
  - 12 mensajes idénticos en 5 minutos
  - Score: 9.2/10
  - Acción sugerida: MUTE 300s o BAN 3600s
  
Moderadores ven ALERT:
  ALERT: Possible spam pattern
    User: A_fp
    Score: 9.2/10
    Evidence: 12 identical messages in 5 min
    Suggested: MUTE 300s
    
Moderator alice ignora (error de sistema)
Moderator bob ejecuta: /warn A "spam detected" "auto-flagged, 1st warning"

A recibe NOTICE: "ADVERTENCIA: Spam de mensajes repetitivos. Por favor modula tu actividad."

Si reincide → /moderate A mute 300
Si reincide 3+ veces → /moderate A ban 86400
```

---

## 🎯 Próximos Pasos

1. **Implementar tipos de datos Rust** para cada comando
2. **Definir grammar/parser** para sintaxis
3. **Implementar máquina de estados** para transiciones válidas
4. **Crear tests** de cada flujo
5. **Integrar con capa de comunicación** (después)

---

## 📝 Notas de Evolución

### Decisión de Diseño: Responsabilidad vs Paternalismo

Esta especificación rechaza el modelo "servidor-papá":
- ✗ "No permitimos que te molesten"
- ✓ "Te damos herramientas para que te protejas"

- ✗ "Notificamos al baneado"
- ✓ "El usuario ignorado no sabe que lo ignoró"

- ✗ "Borramos datos silenciosamente"
- ✓ "Pedimos confirmación antes de cambios importantes"

- ✗ "Moderación siempre punitiva"
- ✓ "Moderación educativa primero, escalada solo si hay resistencia"

- ✗ "Un solo código de ética para todo"
- ✓ "Cada canal define sus reglas, moderadores respetan esa ética"

---

- **Prioridad Fase 0.1**: IGNORE (4 tipos), IDENTIFY, LANG_FILTER, VISIBILITY, PRIVACY STATUS
- **Prioridad Fase 1**: WARN, MODERATE, APPEAL, AUDIT_LOG, RULES, Rate-limiting
- **Prioridad Fase 2+**: Votación en reglas, IA avanzada, IGNORE MAINTENANCE

