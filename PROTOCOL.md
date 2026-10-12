# IRCPilot Protocol Specification v1.0

> **Core Protocol Definition**: The foundation of all communication between clients, servers, and bots in IRCPilot.

---

## 🎯 Design Principles

1. **Backward Compatible with IRC Spirit**: Maintain IRC structure but modernize it
2. **Self-Documenting**: Message type and authority are explicit, not inferred
3. **Granular Queries**: Request only what you need, not full data structures
4. **Multi-Stream Ready**: Support parallel channels (chat, games, media, system)
5. **Privacy-First**: Sensitive data flagged, encrypted separately
6. **Authority-Aware**: Every message declares what type of entity sent it

---

## 📋 Message Format

### Core Structure

```
[AUTHORITY].[SENDER]:[MSGTYPE]::[PAYLOAD]
```

### Authority Types

| Authority | Example | Meaning |
|-----------|---------|---------|
| `System` | `System.welcome` | Internal server bot |
| `Oper` | `Oper.leader` | IRC operator |
| `Verified` | `Verified.alice` | Registered user with fingerprint |
| `Unregistered` | `Unregistered.bob` | Anonymous user (no passphrase) |
| `Bot` | `Bot.trivia_engine` | User-created bot (owned by user) |
| `Relay` | `Relay.discord_bridge` | External relay/bridge |

### Message Types

**Communication:**
- `RoomMsg` - Message in channel
- `UserMsg` - Private message
- `NoticeMsg` - System notice
- `ActionMsg` - /me action

**Responses & Queries:**
- `WhoisReply` - User information
- `ListReply` - Channel list
- `TopicReply` - Channel topic
- `NamesReply` - User list in channel

**Moderation:**
- `WarnMsg` - Warning (user sees it)
- `ModMsg` - Moderation action (all see it)
- `ModPrivate` - Moderation note (mods only)
- `ApprovalMsg` - Decision on appeal/report

**System Events:**
- `SysMsg` - General system message
- `GameMsg` - Game state/event
- `EventMsg` - Community event update
- `NotifyMsg` - Reminder/notification
- `PresenceMsg` - User join/leave/status

**Data Sync:**
- `SyncMsg` - Synchronization data
- `CacheMsg` - Cache invalidation
- `DataMsg` - Structured data transfer

---

## 📨 Complete Message Examples

### Basic Chat
```
Verified.alice:RoomMsg::#general::Hola a todos, ¿qué tal?
```

### Private Message
```
Verified.alice:UserMsg::bob::¿Podemos hablar en privado?
```

### System Message
```
System.welcome:SysMsg::alice::Bienvenido a IRCPilot. Escribe /help para comandos.
```

### Moderation Warning
```
Oper.leader:WarnMsg::bob::Cuidado con el lenguaje. Ver regla #3.
```

### Moderation Action (visible to all)
```
Oper.leader:ModMsg::#gaming::bob fue silenciado 5 minutos por spam.
```

### Bot Response (Game)
```
System.trivia:GameMsg::game_001::{"type":"question","q":"¿Capital de Francia?","opts":["París","Londres","Berlín"],"time":30}
```

### User Join
```
Verified.alice:PresenceMsg::#general::JOIN::2026-10-12T08:30:00Z
```

### Whois Reply (Full)
```
System.server:WhoisReply::bob::{"nick":"bob","fingerprint":"abc123","channels":["#general","#gaming"],"idle":300,"reputation":+15,"warnings":0,"created":"2026-01-15"}
```

---

## 🔍 Query Granularity System

The `/whois<N>` syntax allows requesting specific data fields instead of full records, reducing bandwidth and latency.

### Whois Query Levels

```
/whois <nick>          → Full profile (all fields)
/whois1 <nick>         → Nick only
/whois2 <nick>         → IP mask / fingerprint summary
/whois3 <nick>         → Channels only
/whois4 <nick>         → Reputation only
/whois5 <nick>         → Activity summary (last seen, messages/day)
/whois6 <nick>         → Custom fields (bio, interests, location)
/whois7 <nick>         → Moderation history (warnings, bans, appeals)
/whois8 <nick>         → Achievements/medals
```

### Full Whois Response
```
System.server:WhoisReply::bob::{
  "nick": "bob",
  "fingerprint": "a3b2c1d0e9f8g7h6",
  "identity_age": "2 years",
  "channels": ["#general", "#gaming", "#tech"],
  "idle_seconds": 300,
  "reputation": { "score": 15, "votes": 23 },
  "warnings_active": 0,
  "account_created": "2026-01-15T10:30:00Z",
  "last_seen": "2026-10-12T08:29:00Z",
  "messages_today": 47,
  "bio": "Gamer and Python dev",
  "interests": ["gaming", "python", "anime"],
  "location": "Argentina",
  "achievements": ["First Chat", "Trivia Master"],
  "moderation_history": []
}
```

### Granular Whois Responses

**Whois1 (Nick only):**
```
System.server:WhoisReply::bob::{"nick":"bob"}
```

**Whois2 (Fingerprint summary):**
```
System.server:WhoisReply::bob::{"fingerprint":"a3b2c1d0e9f8g7h6","identity_age":"2 years"}
```

**Whois3 (Channels):**
```
System.server:WhoisReply::bob::{"channels":["#general","#gaming","#tech"]}
```

**Whois4 (Reputation):**
```
System.server:WhoisReply::bob::{"reputation":{"score":15,"votes":23}}
```

**Whois5 (Activity):**
```
System.server:WhoisReply::bob::{"idle_seconds":300,"last_seen":"2026-10-12T08:29:00Z","messages_today":47}
```

**Whois6 (Custom fields):**
```
System.server:WhoisReply::bob::{"bio":"Gamer and Python dev","interests":["gaming","python","anime"],"location":"Argentina"}
```

**Whois7 (Moderation history):**
```
System.server:WhoisReply::bob::{"warnings_active":0,"bans_active":0,"appeals_pending":0,"moderation_history":[]}
```

**Whois8 (Achievements):**
```
System.server:WhoisReply::bob::{"achievements":["First Chat","Trivia Master","Social Butterfly"]}
```

---

## 📊 List Query Granularity

Similar to whois, list queries can be granular:

```
/list                  → Full channel list
/list1                 → Channel names only
/list2                 → User counts only
/list3                 → Topics only
/list4                 → Joined channels (your channels)
/list5                 → Public channels only
/list6                 → Channels matching pattern
```

### Full List Response
```
System.server:ListReply:::#general::{
  "name": "#general",
  "users": 127,
  "topic": "General discussion",
  "created": "2026-01-01T00:00:00Z",
  "visibility": "public",
  "modes": ["+m","+n"],
  "language_filter": "OFF",
  "rules_count": 5
}
```

### Granular List Responses

**List1 (Names only):**
```
System.server:ListReply::0::#general #gaming #tech #music
```

**List2 (Counts only):**
```
System.server:ListReply::0::{"#general":127,"#gaming":45,"#tech":89,"#music":23}
```

**List3 (Topics only):**
```
System.server:ListReply::0::{"#general":"General discussion","#gaming":"Games and fun","#tech":"Technology"}
```

**List4 (Your channels):**
```
System.server:ListReply::you::{"channels":["#general","#gaming","#tech"]}
```

---

## 👥 Names Query Granularity

```
/names #general        → Full user list with roles
/names1 #general       → Nick names only
/names2 #general       → Nick + reputation
/names3 #general       → Nick + status (online/idle/away)
/names4 #general       → Operators only
/names5 #general       → Voiced users only
```

### Full Names Response
```
System.server:NamesReply::#general::{
  "users": [
    {"nick":"alice","role":"voice","reputation":15,"status":"active","idle":0},
    {"nick":"bob","role":"user","reputation":8,"status":"idle","idle":120},
    {"nick":"charlie","role":"op","reputation":45,"status":"away","idle":600}
  ]
}
```

### Granular Names Responses

**Names1 (Nicks only):**
```
System.server:NamesReply::#general::alice bob charlie
```

**Names2 (Nick + reputation):**
```
System.server:NamesReply::#general::{"alice":15,"bob":8,"charlie":45}
```

**Names4 (Operators):**
```
System.server:NamesReply::#general::charlie (1 operator)
```

---

## 🔄 Message Encoding

### Text Format (Human Readable, Backward Compatible)

```
[AUTHORITY].[SENDER]:[MSGTYPE]::[PAYLOAD]
```

Payload can be:
- Plain text (simple messages)
- JSON (structured data)
- Binary (media, encrypted, large data)

### Binary Format (Efficient for Streams)

```
[1 byte: authority_type][2 bytes: sender_len][sender_data]
[1 byte: msgtype_code][2 bytes: payload_len][payload_data]
```

---

## 📡 Multi-Stream Protocol

IRCPilot supports multiple concurrent connections for:
- Main chat (port 6667)
- Games/interactive (port 6668)
- Media/files (port 6669)
- Media/files (port 6670)
- System events (port 6671)

Client can maintain all connections simultaneously.

### Stream Types

**Stream 0: Chat** (6667)
- Main IRC messages
- Private messages
- Moderation messages
- System messages

**Stream 1: Interactive** (6668)
- Games in progress
- Polls/votes
- Real-time events
- Auction/bidding

**Stream 2: Media** (6669)
- File uploads/downloads
- Images, videos, audio
- Large pastes
- Streaming content

**Stream 3: Notifications** (6670)
- Reminders
- Achievements
- System alerts
- Custom notifications

**Stream 4: Sync** (6671)
- Cache synchronization
- Profile updates
- Config changes
- Heartbeat/keepalive

---

## 🔐 Encryption Layers

### Transport Layer (TLS 1.3)
All streams use TLS 1.3 by default.

### Application Layer (E2E Optional)
For sensitive messages:
```
Verified.alice:UserMsg::bob::@e2e::{encrypted_payload}
```

### Fingerprint Validation
Each message includes optional fingerprint signature:
```
Verified.alice:RoomMsg::#general::@sig:abc123::Message content
```

---

## 📝 Legacy IRC Compatibility

IRCPilot maintains backward compatibility with classic IRC:

```
Old IRC:
  :server.irc PRIVMSG #general :Hello world

IRCPilot (new format):
  Verified.alice:RoomMsg::#general::Hello world

Compatibility mode (v1):
  :alice!alice@ircpilot.local PRIVMSG #general :Hello world
```

Server auto-detects client version and responds accordingly.

---

## 🎮 Game/Interactive Message Example

```
System.trivia:GameMsg::game_12345::{
  "type": "question",
  "game_id": "trivia_001",
  "question": "¿Cuál es la capital de Francia?",
  "options": [
    {"id":1,"text":"París"},
    {"id":2,"text":"Londres"},
    {"id":3,"text":"Berlín"},
    {"id":4,"text":"Madrid"}
  ],
  "time_limit": 30,
  "difficulty": "easy",
  "category": "geography"
}
```

---

## ⚙️ System Commands (Non-User)

System commands are issued by System bots or Oper, never directly by users:

```
System.server:SysCmd:::#general::{"cmd":"kick","nick":"bob","reason":"spam"}
System.server:SysCmd::#general::{"cmd":"ban","identity":"abc123","duration":86400}
System.trivia:SysCmd::game_001::{"cmd":"end","winner":"alice","score":150}
```

---

## 📊 Comparison: Old IRC vs IRCPilot Protocol

| Aspect | Old IRC | IRCPilot |
|--------|---------|----------|
| **Sender identification** | `:nick!user@host` | `[Authority].sender` |
| **Message type** | Inferred from context | Explicit `MsgType` |
| **Query granularity** | Full or nothing | `/cmd<N>` for specific fields |
| **Multi-stream** | Single TCP connection | 5+ concurrent streams |
| **Privacy** | No E2E | Optional E2E with `@e2e` tag |
| **Authority clarity** | Ambiguous | Explicit (System/Oper/Verified/Bot) |
| **Extensibility** | Limited | Bot-native, plugin-ready |
| **Backward compat** | N/A | Full compatibility mode |

---

## 🎯 Next Steps

1. **Implement Parser** - Rust parser for both text and binary formats
2. **Stream Manager** - Handle 5+ concurrent connections per client
3. **Message Router** - Route messages to correct stream based on type
4. **Client Adapter** - Support both legacy IRC clients and new IRCPilot clients
5. **Telemetry** - Track message types and bandwidth usage

---

## 📝 Protocol Versioning

- **v1.0** - Current (supports legacy IRC, new authority system, granular queries)
- **v1.1** (planned) - Binary compression, improved E2E
- **v2.0** (future) - WebSocket support, HTTP/3 streaming

---

**Protocol is versioned with server. Client negotiates compatible version at handshake.**

```
Client: PROTOCOL_VERSION 1.0
Server: PROTOCOL_VERSION 1.0 OK
```

