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
- Hoy: usuario anónimo = oveja sin protección en rebañosno siempre bien intencionados.
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

## 📊 Estado Actual del Proyecto

| Aspecto | Estado | Notas |
|---------|--------|-------|
| **Repositorio** | ✅ Creado | `drthands-apps/IRCPilot` |
| **Licencia** | ✅ AGPL-3.0 | Elección correcta para software de red comunitario |
| **Documentación** | 🟡 En Inicio | README mínimo. Necesita PHILOSOPHY, ROADMAP, ARCHITECTURE |
| **Base de Código** | 🔴 No iniciado | Pendiente: seleccionar IRC Rust base e inicializar |
| **Milestone Inicial** | 🟡 Planeando | Fase 1: Core + Seguridad + Moderación |

---

## 🗺️ Fases del Proyecto (Roadmap Provisional)

### **Fase 0: Fundamentos (Actual - 2-4 semanas)**
- [ ] Documentación: PHILOSOPHY.md, ARCHITECTURE.md
- [ ] Seleccionar base IRC Rust (investigar opciones)
- [ ] Setup inicial: `Cargo.toml`, estructura de módulos
- [ ] Plan de implementación detallado

**Deliverables:**
- Documentación clara y actualizable
- Estructura de proyecto lista
- Primer commit con scaffolding

---

### **Fase 1: Core + Seguridad (4-8 semanas)**
- [ ] Servidor IRC funcional (puerto, clientes básicos)
- [ ] Encriptación E2E (TLS/DTLS + protocolo encriptado)
- [ ] Sistema de anonimato (nicknames, no IP tracking, tor-compatible)
- [ ] Almacenamiento seguro de datos

**Deliverables:**
- Servidor funcional accesible vía mIRC/clientes estándar
- Conexión encriptada end-to-end
- Identidad anónima protegida

---

### **Fase 2: Moderación Avanzada (6-10 semanas)**
- [ ] Sistema de moderación bidireccional (ignore bilateral)
- [ ] Reglas de canal inmutables y auditables
- [ ] Apelación de bans/silences
- [ ] Detección de spam/abuso con ML (opcional fase posterior)
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

| Capa | Tecnología | Razón |
|------|-----------|-------|
| **Lenguaje Core** | Rust | Seguridad, performance, concurrencia |
| **Base IRC** | `irc-proto` o fork de proyecto existente | No reinventar rueda |
| **Encriptación** | TLS 1.3 + custom E2E layer | Standard + extra privacy |
| **Base Datos** | PostgreSQL + Redis | Confiable, escalable, ACID |
| **API/Comunicación** | Binary protocol + WebSocket | Eficiente + accesible |
| **Deployment** | Docker + Kubernetes | Escalable y reproducible |
| **Testing** | Criterion + Tokio test suite | Performance + confiabilidad |

---

## 🚧 Decisiones Pendientes

1. **¿Qué proyecto IRC existente usar como base?**
   - [ ] Investigar: `irsc`, `irc-proto`, `unrealircd` (Rust port?), alternativas
   - [ ] Criterios: Modular, sin legacy pesado, documentado

2. **¿Encriptación: estándar o custom?**
   - [ ] TLS standard + layer E2E custom para anonimato
   - [ ] O protocolo completamente custom?

3. **¿Compatibilidad total con clientes IRC estándar?**
   - [ ] Sí (mIRC, weechat, etc.) → más trabajo inicial
   - [ ] Parcial + cliente propio → control total

4. **¿Federación desde el inicio o después?**
   - [ ] Desde fase 1 (complejo)
   - [ ] Después de core funcional (más simple)

---

## 📝 Documentos Relacionados

- **PHILOSOPHY.md** (TODO): Expansión filosófica detallada
- **ARCHITECTURE.md** (TODO): Diagrama técnico y diseño de módulos
- **CONTRIBUTING.md** (TODO): Guía para colaboradores
- **SECURITY.md** (TODO): Modelo de seguridad y privacidad
- **ROADMAP.md** (TODO): Versión expandida de fases con issues

---

## 💬 Notas para Sesiones Futuras

### Ayer (11 Oct 2026)
- Creado repo, licencia AGPL-3.0
- Discusión sobre filosofía y stack (Rust)
- Decisión: base IRC existente + features nuevas

### Hoy (12 Oct 2026)
- Creado CONTEXT.md (este archivo)
- Próximo: Elegir base IRC, crear ARCHITECTURE.md

### Próximas sesiones
- Investigar proyectos IRC en Rust
- Escribir ARCHITECTURE.md con decisiones técnicas
- Setup inicial de `Cargo.toml` y scaffolding
- Crear issues en GitHub por fase

---

## ✏️ Cómo Mantener Este Documento

1. **Al inicio de cada sesión**: Lee este archivo para contexto
2. **Al finalizar cada sesión**: Actualiza secciones relevantes:
   - Estado Actual
   - Decisiones Pendientes (marca lo resuelto)
   - Notas para Sesiones Futuras
3. **Cada fase completada**: Mueve checklist a ✅ completado
4. **Cambios de rumbo**: Documenta por qué y qué cambia

---

## 🔗 Links Útiles

- Repositorio: https://github.com/drthands-apps/IRCPilot
- Licencia AGPL-3.0: https://www.gnu.org/licenses/agpl-3.0.html
- IRC Protocol RFC 1459: https://tools.ietf.org/html/rfc1459
- Rust async: https://tokio.rs/

---

**Este documento es la fuente de verdad sobre IRCPilot. Si algo confunde en el código o decisiones, aquí encontrará respuestas.**

