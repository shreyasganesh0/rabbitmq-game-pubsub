# Distributed Multiplayer Game Server (Go & RabbitMQ)

This project is a high-performance backend for a real-time, text-based multiplayer game, built entirely in Go. It demonstrates a complex distributed systems architecture by using RabbitMQ as the core messaging and transport layer, decoupling the game server from its clients.

Instead of a monolithic architecture, this system is composed of an authoritative game server (cmd/server) and multiple clients (cmd/client) that communicate asynchronously over a message bus.

## Core Architectural Concepts

1. Authoritative Server: The cmd/server process acts as the single source of truth. It initializes and maintains the central GameState (internal/gamelogic/gamestate.go).

2. Decoupled Clients: Clients (cmd/client) do not connect to the server directly. Instead, they publish their intended actions (e.g., "move", "attack") to RabbitMQ queues.

3. Asynchronous Command Processing: The server consumes player commands from a dedicated queue, processes them against the game logic (war.go, move.go), and updates the central game state.

4. Pub/Sub for State Broadcasting: When the game state changes, the server publishes updates to a RabbitMQ "fan-out" exchange. All connected clients are subscribed to this exchange and receive the state updates in real-time, ensuring all players are synchronized.

5. Containerized Deployment: The entire application (client and server) is containerized using a multi-stage Dockerfile, allowing for easy, isolated, and repeatable deployments.

## Technical Highlights

internal/gamelogic: The stateful, core game engine. Manages player spawning, movement, combat, and game-over conditions.

internal/pubsub: An abstraction layer that wraps the RabbitMQ client, providing clean Publish and Subscribe interfaces to the rest of the application.

internal/routing: Defines the shared data structures and routing keys used for client-server communication.

rabbit.sh & multiserver.sh: Utility scripts for spinning up the RabbitMQ broker and multiple server instances for testing.

## How to Run (via Docker)

Start RabbitMQ:
```
./rabbit.sh
```

Build the Docker Image:
```
docker build -t rabbitmq-game .
```

Run the Server:
```
docker run --rm --network=host rabbitmq-game server
```

Run a Client (in a new terminal):
```
docker run --rm --network=host rabbitmq-game client
```
