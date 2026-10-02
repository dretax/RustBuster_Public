### RustBuster2016 Plugin Documentation

Public API documentation for RustBuster2016 plugin developers, split by which side of the
client/server boundary it runs on.

### Layout

```
docs/
  README.md        <- you are here
  Client/           <- public client-side API (runs inside the game process)
    README.md
    Guide/
    Classes/
    Hooks/
  Server/           <- public server-side API (runs inside the RustBuster server process)
    README.md
    Classes/
```

### Where to start

- **Writing a client plugin** (runs in the game, reacts to local gameplay): start with
  [`Client/README.md`](Client/README.md).
- **Writing a server plugin** (runs on the server, has authority over the game world): start with
  [`Server/README.md`](Server/README.md).

Both sides use the same general plugin conventions (a `Module`/`RustBusterPlugin`-derived class with
`Initialize()`/`DeInitialize()`, subscribing to static `Hooks` events with `+=`/`-=`), but the available
classes/hooks are entirely different, since they run in two different processes with different
responsibilities.
