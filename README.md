# CommunicationModEnhanced
Slay the Spire mod that provides a socket protocol for allowing another process to control the game.

## Requirements

- Slay the Spire
- ModTheSpire (https://github.com/kiooeht/ModTheSpire)
- BaseMod (https://github.com/daviscook477/BaseMod)

## Setup

1. Copy `CommunicationModEnhanced.jar` to your ModTheSpire mods directory.
2. Run ModTheSpire with CommunicationModEnhanced enabled.
3. Edit your newly-created SpireConfig file with the command you want to use with CommunicationModEnhanced (see https://github.com/kiooeht/ModTheSpire/wiki/SpireConfig for the location of your config file). A socket-based config should look like this:

```properties
command=python C\:\\Path\\To\\Script\\main.py
transport=socket
socketHost=127.0.0.1
socketPort=0
maxInitializationTimeout=120
asyncInitialization=true
```

`socketPort=0` lets CommunicationModEnhanced pick an open localhost port automatically.

## What does this mod do?

CommunicationModEnhanced launches a specified process and communicates with that process over a localhost TCP socket.

The startup flow is:

1. CommunicationModEnhanced opens a TCP server on `socketHost:socketPort`.
2. CommunicationModEnhanced launches the external process from `command`.
3. CommunicationModEnhanced passes the chosen socket address to the external process through environment variables:
   - `COMMUNICATIONMOD_TRANSPORT=socket`
   - `COMMUNICATIONMOD_HOST`
   - `COMMUNICATIONMOD_PORT`
   - `COMMUNICATIONMOD_CONNECT_TIMEOUT_SECONDS`
4. The external process connects back to CommunicationModEnhanced and sends `ready`.
5. CommunicationModEnhanced starts sending game states after the `ready` handshake completes.

All protocol messages are still newline-delimited:

- CommunicationModEnhanced sends one JSON game-state payload per line.
- The external process sends one command string per line.
- Error payloads still look like:

```json
{"error":"Error message","ready_for_command":true}
```

The game-state payload and command set are unchanged from the original protocol.

## Debugging

- `stdout` and `stderr` from the external process are no longer part of the protocol.
- CommunicationModEnhanced redirects both streams to `communication_mod_errors.log`.
- This means debug logging, third-party library output, and `debugpy` warnings will not be misread as game commands.

For debugger-attached runs such as `debugpy --wait-for-client`, keep `asyncInitialization=true`. CommunicationModEnhanced will keep waiting for the socket connection and `ready` signal without requiring the bot to speak on `stdout`.

## Known limitations

- You must still manually edit the mod's config file to set the external process command.
- Communication Mod has not been tested without fast mode on.
- This fork no longer supports the old stdio protocol; socket transport is required.
