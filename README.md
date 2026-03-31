# Network_multplayer_game-cubes

Multiplayer “cubes/dice” game with a **Go server** and a **Python client**. The project focuses on network communication, message parsing, and a simple game/state workflow.

## Repository structure

- `GameServer/` — Go server implementation
  - `GameServer/internal/main.go` — application entrypoint
  - Modules (conceptually):
    - **Command Processing** — handles user/client commands
    - **Models** — game/player/message data structures, lists/collections, state machine
    - **Network** — sending/receiving messages
    - **Parser** — converts network messages ↔ internal objects
    - **Server Listen** — main loop that listens for messages and dispatches processing
- `GameClient/` — Python client implementation

Main documantation is saved here [docs/UPS_semestralka_dokumentace_ZS_23_2420(1).pdf](https://github.com/cernyfili/FAV_2324_UPS_Network_multplayer_game-cubes/blob/master/docs/UPS_semestr%C3%A1lka_dokumentace_ZS_23_24%20(1).pdf)

Documentation for server lives under `GameServer/docs/`:
- `docs/server_stavy.md` — protocol + commands + finite automata diagrams
- `docs/komunikacni_protokol.md` — high-level protocol notes (client/server events)

## Tech stack

- **Go (target: Go 1.23)** for the server
- **Python (3.1+)** for the client
- Build/run via **make**
- Libraries mentioned in docs:
  - `stateless` (state machine)
  - `github.com/pkg/errors`
  - `github.com/sirupsen/logrus`

## Concurrency model (server)

The server uses Go concurrency primitives:
- per-client handling is designed around **goroutines**
- asynchronous processing of network messages via goroutines
- mutexes (`sync.Mutex`) can be used where shared state must be protected (see `docs/wiki/multhreading_go.md`)

## Network protocol overview

### Message format

Messages are described as:

- `stamp;command_id;timestamp;{player_nickname};{args...}`

Example (see `GameServer/docs/server_stavy.md`):
- `KIVUPS01... 2024-12-31 15:30:00.000000{nickname}{}\n`

Parameters are JSON-like:
- single params example: `{"gameName":"Game3", "maxPlayers":"3"}`
- list params are encoded into a single field (see `docs/server_stavy.md` for list formats like `gameList`, `playerList`, `gameData`, `cubeValues`)

### Commands (current, summary)

Client → Server commands (selected):
- `ClientLogin` (`CommandID: 1`) — nickname in header
- `ClientCreateGame` (`CommandID: 2`) — params: `gameName`, `maxPlayers`
- `ClientJoinGame` (`CommandID: 3`) — params: `gameName`
- `ClientStartGame` (`CommandID: 4`)
- `ClientRollDice` (`CommandID: 5`)
- `ClientSelectedCubes` (`CommandID: 61`) — params: `cubeValues` list
- `ClientLogout` (`CommandID: 7`)
- `ClientReconnect` (`CommandID: 8`)

Server → Client responses / updates include:
- `ResponseServerSuccess` (`CommandID: 30`)
- `ResponseServerError` (`CommandID: 32`, param: `message`)
- `ResponseServerGameList` (`CommandID: 33`, `gameList`)
- `ServerUpdateGameList` (`CommandID: 44`, `gameList`) — broadcast
- `ServerUpdatePlayerList` (`CommandID: 45`, `playersList`) — broadcast
- `ServerUpdateGameData` (`CommandID: 43`, `gameData`) — broadcast
- `ServerStartTurn` (`CommandID: 49`) — single client
- `ServerPingPlayer` (`CommandID: 50`) — keepalive / reachability checks

For the full list, formats, and the player finite automata, see:
- `GameServer/docs/server_stavy.md`

## How to run

### Server

1. Install **Go 1.23** and **make**
2. Clone the repository
3. Go to the server folder:
   - `cd GameServer`
4. Run:
   - `make`

### Client

1. Install **Python 3.1+** and **make**
2. Go to the client folder:
   - `cd GameClient`
3. Run:
   - `make`

> Note: exact make targets depend on the Makefiles in `GameServer/` and `GameClient/`.
