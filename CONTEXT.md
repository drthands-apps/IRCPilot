# IRCPilot - Contexto y Guía de Proyecto

> **Documento vivo**: Este archivo es la brújula del proyecto. Se actualiza conforme evolucionan los objetivos y aprendizajes. Léelo primero en cada sesión.

---

## 🎯 Misión Fundamental

Devolver a la comunidad de internet lo que siempre fue suyo: **un espacio colaborativo, seguro, democrático y divertido** donde la privacidad no es mercancía y el conocimiento es libre.

---

## 🏛️ Los 6 Pilares Filosóficos

### 1. **Deuda Comunitaria**
- Grandes corporaciones se apropiaron del conocimiento colectivo anónimo de décadas.
- IRCPilot devuelve ese valor: código abierto, sin cercos, accesible a todos.
- Cada línea es un agradecimiento a quienes construyeron internet colaborativamente.

### 2. **Democratización de Sistemas**
- IRC funcionaba bien, pero tenía defectos conceptuales y de adaptación moderna.
- No reinventamos la rueda: mejoramos el diseño, corregimos errores, lo hacemos escalable.
- Arquitectura actualizable sin perder esencia: evoluciona sin morir.

### 3. **Protección del Usuario Anónimo**
- Hoy: usuario anónimo = oveja sin protección en rebaños no siempre bien intencionados.
- Mañana: ciudadano de una polis griega con derechos reales.
  - Defensa contra acoso, spam, abuso.
  - Control total sobre datos propios.
  - Moderación justa, transparente, impugnable.

### 4. **Internet No Mercantilista**
- No todo tiene precio. No todo se vende.
- La privacidad ≠ dato a explotar.
- La intimidad de cada usuario es **inviolable**, no activo comercial.

### 5. **Diversión y Sostenibilidad Social**
- Internet fue divertido. Lo volvemos divertido.
- Juegos, actividades, comunidad, intercambio de contenidos.
- Sostenibilidad: features que mantengan viva la plataforma sin explotación.

### 6. **Base Sólida para Futuro**
- Código limpio, arquitectura escalable, documentación clara.
- Permite que el proyecto evolucione sin colapsar.
- Preparado para incorporar tecnologías futuras.

---

## 🏗️ Arquitectura Estratégica (CRÍTICA)

### Dos Capas Independientes: Comunicación e Interpretación

El servidor se divide en **dos sistemas fundamentales que DEBEN ser independientes**:

#### **1. Sistema de Interpretación (La Lógica)**
- Procesa comandos, ejecuta acciones, gestiona estado
- **Agnóstico respecto a si los datos están encriptados o no**
- Punto de entrada: comandos estructurados (parseados)
- No toca encriptación, no hace I/O de red
- Es el "cerebro" del servidor

**Características:**
- Modular, testeable, predecible
- Puede ser testeado sin red
- Escalable: múltiples instancias procesando comandos en paralelo
- API clara: `Command → Action → State Mutation`

#### **2. Sistema de Comunicación (La Tubería)**
- Gestiona conexiones de red, encriptación, transmisión
- Lee bytes de la red, los descifra (si aplica)
- Los convierte a `Command` estructurado
- Los envía al sistema de interpretación
- Recibe respuestas y las envía encriptadas de vuelta

**Características:**
- Abstracción sobre el medio de transmisión
- Puede ser TLS, encriptación custom, WebSocket, lo que sea
- El servidor nunca descifra contenido privado (solo metadatos mínimos)
- Frontera clara con la lógica

### Por Qué Esta Arquitectura Es Fuerte

✅ **Privacidad verdadera**: El servidor procesa sin ver contenido  
✅ **Escalabilidad**: Interpretación en workers, comunicación en otro tier  
✅ **Testabilidad**: Testeas lógica sin mocks de red  
✅ **Flexibilidad**: Cambias encriptación sin tocar lógica  
✅ **Modularidad**: Cada capa es independiente  

### Flujo de Datos

```
[Cliente Encriptado] 
    ↓ (red)
[Capa Comunicación: Desencriptación]
    ↓ (byte → Command)
[Queue/Bus de Comandos]
    ↓
[Capa Interpretación: Procesa comando]
    ↓ (Action, State mutations)
[Respuesta estructurada]
    ↓ (Command → Bytes)
[Capa Comunicación: Encriptación]
    ↓ (red)
[Cliente recibe respuesta encriptada]
```

---

## 📊 Estado Actual del Proyecto

| Aspecto | Estado | Notas |
|---------|--------|-------|
| **Repositorio** | ✅ Creado | `drthands-apps/IRCPilot` |
| **Licencia** | ✅ AGPL-3.0 | Elección correcta para software de red comunitario |
| **Filosofía** | ✅ Documentada | 6 pilares claros |
| **Arquitectura** | ✅ Definida | Dos capas: Comunicación e Interpretación (agnósticas) |
| **Documentación Técnica** | 🟡 En Inicio | Necesita ARCHITECTURE.md detallado, specs de protocolo |
| **Base de Código** | 🔴 No iniciado | Pendiente: Fase 0 - scaffolding en Rust |
| **Milestone Inicial** | 🟡 Planeando | Fase 0.1: Motor de comandos básico + testeable |

---

## 🗺️ Fases del Proyecto (Roadmap Revisado)

### **Fase 0: Fundamentos (Actual - 2-4 semanas)**
- [ ] Documentación arquitectónica: ARCHITECTURE.md
- [ ] Especificación de protocolo de comandos
- [ ] Setup inicial: `Cargo.toml`, estructura de módulos
- [ ] Decisión: ¿encriptación TLS estándar o custom?

**Deliverables:**
- Arquitectura documentada y clara
- Estructura de proyecto lista
- Primer commit con scaffolding

---

### **Fase 0.1: Motor de Comandos Mínimo (1-2 semanas)**
- [ ] Sistema de interpretación base (SIN comunicación de red aún)
- [ ] Parser de comandos escalable
- [ ] Sistema de acciones (mutations)
- [ ] Tests exhaustivos del motor

**Deliverables:**
- Motor de comandos funcional y testeado
- API clara para agregar nuevos comandos
- Cero dependencias de red

**Por qué primero esto:**
- Es lo fundamental, agnóstico a transporte
- Se puede testear sin red
- Es la base sobre la que construye todo lo demás

---

### **Fase 1: Comunicación Base (2-3 semanas)**
- [ ] Capa de comunicación (abstracción de red)
- [ ] Encriptación TLS 1.3 base
- [ ] Parser IRC → `Command` estructurado
- [ ] Cliente test simple (telnet o similar)
- [ ] Integración motor + comunicación

**Deliverables:**
- Servidor escucha en puerto, acepta conexiones TLS
- Cliente simple puede conectarse y enviar comandos
- Respuestas encriptadas vuelven al cliente

---

### **Fase 2: Moderación Avanzada (6-10 semanas)**
- [ ] Sistema de moderación bidireccional (ignore bilateral)
- [ ] Reglas de canal inmutables y auditables
- [ ] Apelación de bans/silences
- [ ] Auditoría de acciones de moderación

**Deliverables:**
- Moderadores y usuarios empoderados
- Transparencia total en sanciones
- Menos abuso, más justicia

---

### **Fase 3: Contenido Interactivo (6-8 semanas)**
- [ ] Sistema de compartición de archivos (encriptado)
- [ ] Juegos internos (trivia, carreras, minijuegos IRC)
- [ ] Actividades comunitarias (eventos, competiciones)
- [ ] Integración con bots de utilidad

**Deliverables:**
- Plataforma divertida y atractiva
- Engagement comunitario alto
- Diferenciación vs IRCs tradicionales

---

### **Fase 4: Escalabilidad y Federación (8-12 semanas)**
- [ ] Soporte multi-servidor (federación)
- [ ] Base de datos distribuida
- [ ] Replicación segura de canales/datos
- [ ] Load balancing y redundancia

**Deliverables:**
- Infraestructura resiliente y escalable
- Capacidad de crecer sin perder identidad
- Preparado para millones de usuarios

---

### **Fase 5: Clientes y Accesibilidad (Paralelo a otras fases)**
- [ ] Plugin/cliente mIRC nativo
- [ ] Cliente web (Svelte/React)
- [ ] Cliente mobile (Android Kotlin via FenixIRC, iOS)
- [ ] Accesibilidad (WCAG, screen readers)

**Deliverables:**
- Acceso desde cualquier dispositivo y cliente
- UX moderna pero compatible con legacy

---

## 🛠️ Stack Tecnológico

| Capa | Componente | Tecnología | Razón |
|------|-----------|-----------|-------|
| **Lenguaje Core** | Todo | Rust | Seguridad, performance, concurrencia |
| **Motor de Comandos** | Interpretación | Trait-based + async | Modular, escalable, testeable |
| **Comunicación** | Red | `tokio` + `bytes` | Async I/O eficiente |
| **Encriptación** | Transporte | TLS 1.3 (rustls) | Standard seguro |
| **Protocolo Privado** | E2E (opcional) | Custom (simple) | Privacidad sin overhead |
| **Base Datos** | Persistencia | PostgreSQL + Redis | ACID + caché |
| **Serialización** | Comandos | Binario compacto o JSON | Eficiencia + facilidad |
| **Testing** | QA | Criterion + tokio::test | Confiabilidad |

---

## 🚧 Decisiones Pendientes (CRÍTICAS)

1. **Protocolo de comandos: IRC estándar o custom?**
   - IRC: Compatible con clientes estándar pero limita privacidad
   - Custom: Total control pero requiere cliente propio
   - **Propuesta**: Capa de abstracción que traduce IRC → comandos internos

2. **¿Encriptación TLS o E2E?**
   - TLS estándar: servidor descifra, cliente confía
   - E2E custom: servidor nunca ve contenido
   - **Propuesta**: TLS para autenticación + capa E2E opcional para mensajes

3. **¿Motor de comandos con base de datos o en memoria?**
   - En memoria: Fast pero no persiste
   - Con DB: Persiste pero más lento
   - **Propuesta**: Híbrido (caché en memoria, sync a DB)

4. **¿Qué comandos iniciales implementar?**
   - Mínimo IRC: JOIN, PART, PRIVMSG, QUIT
   - Plus: MODERATOR, RULES, APPEAL, IGNORE
   - **Propuesta**: Empezar con 5-10 comandos core, agregar después

---

## 📝 Documentos Relacionados

- **ARCHITECTURE.md** (TODO): Arquitectura técnica detallada + diagramas
- **PROTOCOL.md** (TODO): Especificación del protocolo de comandos
- **PHILOSOPHY.md** (TODO): Expansión filosófica detallada
- **CONTRIBUTING.md** (TODO): Guía para colaboradores
- **SECURITY.md** (TODO): Modelo de seguridad y privacidad
- **ROADMAP.md** (TODO): Versión expandida de fases con issues

---

## 💬 Notas para Sesiones Futuras

### Sesión 1 (11 Oct 2026)
- Creado repo, licencia AGPL-3.0
- Discusión sobre filosofía y stack (Rust)
- Decisión: base IRC existente + features nuevas

### Sesión 2 (12 Oct 2026)
- Creado CONTEXT.md (filosofía + roadmap inicial)
- **BREAKTHROUGH**: Arquitectura de dos capas (Comunicación e Interpretación)
- Decisión: Motor de comandos primero, comunicación después
- Próximo: ARCHITECTURE.md detallado, specs de protocolo

### Próximas sesiones
- Escribir ARCHITECTURE.md con diagrama de capas
- Especificar protocolo de comandos (qué es un `Command`)
- Setup inicial: `Cargo.toml` + estructura básica
- Implementar Fase 0.1: Motor de comandos testeable
- Crear issues en GitHub por componente

---

## ✏️ Cómo Mantener Este Documento

1. **Al inicio de cada sesión**: Lee este archivo para contexto
2. **Al finalizar cada sesión**: Actualiza secciones relevantes:
   - Estado Actual
   - Decisiones Pendientes (marca lo resuelto)
   - Notas para Sesiones Futuras
3. **Cambios de arquitectura**: Documenta en sección "Arquitectura Estratégica"
4. **Cambios de rumbo**: Explica por qué en "Notas para Sesiones Futuras"

---

## 🔗 Links Útiles

- Repositorio: https://github.com/drthands-apps/IRCPilot
- Licencia AGPL-3.0: https://www.gnu.org/licenses/agpl-3.0.html
- IRC Protocol RFC 1459: https://tools.ietf.org/html/rfc1459
- Rust async (Tokio): https://tokio.rs/
- Rustls (TLS): https://docs.rs/rustls/

---

**Este documento es la fuente de verdad sobre IRCPilot. Si algo confunde en el código o decisiones, aquí encontrará respuestas.**
