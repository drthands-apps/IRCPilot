# IRCPilot - Especificación de Comandos y Semántica

> **Documento de Diseño**: Define los comandos del sistema, su semántica y diferencias con IRC clásico. Es la base para el motor de interpretación.

---

## 🎯 Principio de Diseño

**No copiamos IRC. Lo rediseñamos desde cero respetando su usabilidad pero cumpliendo nuestros objetivos de privacidad, seguridad y protección del usuario anónimo.**

Los comandos son la "interfaz de derechos": cada comando refleja un derecho del usuario y una regla de moderación.

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

### Categoría F: Reglas y Gobernanza
- `RULES` - Ver/establecer reglas de canal
- `VOTE` - Sistema de votación (futuro)

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
- **Visibilidad**: `<nick>` NO sabe que lo ignoras

**Caso de uso**: Usuario molesto en una charla puntual, pero puede que vuelva.

---

#### Tipo 2: `IGNORE PERMANENT`
```
/ignore <nick> permanent
```
- **Alcance**: Todos tus nicknames en IRCPilot (vinculado a identidad)
- **Duración**: Indefinido (hasta `UNIGNORE`)
- **Efecto**: No ves mensajes de `<nick>` en ningún contexto
- **Bilateral**: NO
- **Visibilidad**: `<nick>` NO sabe que lo ignoras

**Caso de uso**: Usuario permanentemente molesto que querés bloquear en todo IRCPilot.

**Nota**: El ignore se vincula a la identidad del usuario, no al nick (ver sección IDENTIFY).

---

#### Tipo 3: `IGNORE BILATERAL` (MUTUAL BLOCK)
```
/ignore <nick> bilateral
```
- **Alcance**: Bidireccional completo
- **Duración**: Indefinido
- **Efecto**: 
  - Ni tú ves mensajes de `<nick>`
  - Ni `<nick>` ve mensajes tuyos
  - Si estáis en un canal, ni vuestros nicks aparecen en mensajes del otro
  - Sistema privado: no se entera mutuamente
- **Bilateral**: SÍ (ambos quedan separados)
- **Visibilidad**: Ambos saben que están mutuamente ignorados

**Caso de uso**: Dos usuarios que no quieren interactuar en absoluto. "Separación amigable" pero completa.

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

## 🚨 Moderación Autonóma y Apelación

### Sistema de Moderación Automática (IA/Heurística)

```
/report <nick> <razón>
```

**Proceso:**
1. Usuario reporta conversación o nick
2. Sistema analiza automáticamente:
   - Patrones de spam (mensajes repetitivos)
   - Hate speech (palabras clave)
   - Acoso (menciones agresivas repetidas)
   - Flood (demasiados mensajes en poco tiempo)
3. Si score > umbral, marca para revisión humana o acción automática
4. **Importante**: Las conversaciones analizadas SON PRIVADAS (encriptadas) pero el servidor puede hacer análisis básicos sin descifrar

**Diseño de privacidad:**
- Análisis con hash de palabras, no contenido plano
- No se almacenan transcripciones
- Solo metadatos (quién, cuándo, score)

---

### Sistema de Apelación (Justicia Descentralizada)

Si un usuario es baneado, silenciado o reportado, puede apelar:

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

---

### `MODERATE` - Acciones de Moderador

```
MODERATE <nick> <acción> <duración> [razón]
```

Acciones:
- `MUTE <segundos>` - Silenciar por X segundos
- `KICK` - Expulsar del canal (no baneo permanente)
- `BAN <identidad> <segundos>` - Baneo temporal a identidad
- `UNBAN <identidad>` - Desbanear

**Importante:**
- Baneo es por IDENTIDAD (fingerprint), no por nick/IP
- Cada acción queda auditada
- Moderador debe dar razón (en auditoría)

---

### `AUDIT_LOG` - Ver Auditoría

```
AUDIT_LOG [#canal|nick] [last <N> lines]
AUDIT_LOG search <pattern>
```

**Tipos de eventos auditados:**
- Baneo/silenciamiento
- Cambios de reglas
- Reports
- Apelaciones
- Acciones de moderador

**Restricciones:**
- Usuario normal: solo ve sus propios eventos
- Moderador: ve eventos de su canal
- Admin: ve todo
- Timestamps encriptados con privacidad

---

## 📊 Tabla Comparativa: IRC Clásico vs IRCPilot

| Aspecto | IRC Clásico | IRCPilot |
|---------|-------------|----------|
| **IGNORE** | Unidireccional, básico | 4 tipos (session, permanent, bilateral, appeal) |
| **Identidad** | Nick + IP | Fingerprint criptográfico |
| **Privacidad Contenido** | Servidor ve todo | Mensajes E2E encriptados |
| **Moderación** | Manual, opaca | Automática + apelación transparente |
| **Reglas** | Informales | Formales, inmutables, auditadas |
| **Lenguaje** | Un nivel | Múltiples filtros por edad |
| **Auditoría** | Ninguna | Completa y accesible |
| **Apelación** | No existe | Sí, con votación |
| **Visibilidad Presencia** | Siempre público | Configurable (PUBLIC/STEALTH/GHOST) |
| **Baneos** | Por IP (falsificable) | Por identidad (difícil falsificar) |

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

### Flujo 2: Abuso y Apelación

```
Usuario A reporta a Usuario B por acoso
  → Sistema analiza: 50 menciones agresivas en 1 hora
  → Score > umbral → Baneo automático por 24 horas
  → B recibe NOTICE: "Estás baneado por acoso. Puedes apelar con /appeal"
  → B ejecuta: /appeal A "fue en broma"
  → Panel de moderadores revisa evidencia
  → Votación: 3 mantienen baneo, 1 lo levanta
  → Baneo se sostiene
  → B recibe AUDIT_LOG con razonamiento
```

---

### Flujo 3: Mutual Block (Separación Completa)

```
Usuario A y B tuvieron discusión tóxica
  → A ejecuta: /ignore B bilateral
  → Sistema obtiene confirmación de ambos
  → Resultado:
    - A no ve mensajes de B
    - B no ve mensajes de A
    - Si ambos están en #general:
      - A ve: "Usuario anónimo dijo algo" (nick de B oculto)
      - B ve: "Usuario anónimo dijo algo" (nick de A oculto)
    - En privado: PRIVMSG rechazado con "Usuario bloqueado"
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

- **Este documento es vivo**: Los comandos evolucionarán
- **Prioridad Fase 0.1**: IGNORE, IDENTIFY, LANG_FILTER, VISIBILITY
- **Prioridad Fase 1**: MODERATE, APPEAL, AUDIT_LOG, RULES
- **Prioridad Fase 2+**: Votación en reglas, IA avanzada

