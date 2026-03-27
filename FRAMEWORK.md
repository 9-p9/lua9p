# lua9p — 9cog Framework Role

## Role: Lightweight 9P Scripting Client

lua9p provides a pure-Lua 9P client for scripting interactions with the
9cog AI filesystem, enabling embedded and lightweight environments to
access framework services.

## Framework Integration

### AI Filesystem Client Helpers

```lua
-- 9cog framework helpers for lua9p
local p9 = require '9p'

-- Connect to AI filesystem
local function ai_connect(host, port)
    -- Create TCP connection and attach to /ai
    local conn = p9.newconn(read_fn, write_fn)
    p9.attach(conn, os.getenv("USER") or "anonymous", "/ai")
    return conn
end

-- Chat with AI
local function ai_chat(conn, query)
    local ctl = p9.walk(conn, conn.rootfid, nil, "sessions/new/ctl")
    p9.open(conn, ctl, p9.OWRITE)
    p9.write(conn, ctl, 0, query)
    p9.clunk(conn, ctl)

    local hist = p9.walk(conn, conn.rootfid, nil, "sessions/1/history")
    p9.open(conn, hist, p9.OREAD)
    local data = p9.read(conn, hist, 0, 8192)
    p9.clunk(conn, hist)
    return data
end

-- Query knowledge graph
local function ai_knowledge(conn, pattern)
    local query = p9.walk(conn, conn.rootfid, nil, "knowledge/query")
    p9.open(conn, query, p9.ORDWR)
    p9.write(conn, query, 0, pattern)
    local result = p9.read(conn, query, 0, 8192)
    p9.clunk(conn, query)
    return result
end

-- Get topology status
local function ai_topology(conn)
    local lc = p9.walk(conn, conn.rootfid, nil, "topology/lifecycle")
    p9.open(conn, lc, p9.OREAD)
    local status = p9.read(conn, lc, 0, 4096)
    p9.clunk(conn, lc)
    return status
end
```

## Architecture

```
┌────────────────────────────┐
│  Lua application           │
│  (embedded, IoT, scripts)  │
├────────────────────────────┤
│  lua9p (this repo)         │
│  9P2000 client library     │
│  + framework helpers       │
├────────────────────────────┤
│  TCP / Unix socket         │
├────────────────────────────┤
│  aifs server (go9p)        │
│  AI filesystem             │
└────────────────────────────┘
```

## Key Files

| File | Purpose |
|------|---------|
| `9p.lua` | Core 9P protocol client |
| `testclient.lua` | Example usage (reference for framework helpers) |

## Unique Contribution

lua9p's flexible I/O callback design (`newconn(read, write)`) makes it
adaptable to any transport mechanism, enabling AI filesystem access from:
- Embedded Lua environments (games, editors, IoT)
- Redis/Nginx Lua scripting
- LuaJIT applications
- Plan 9 Lua interpreters

## See Also

- [9cog Framework Architecture](https://github.com/9cog/9fs9rc/blob/main/framework/ARCHITECTURE.md)
