# Unreal Agent Bridge — Random Stylized Forest

## Assignment choice

- **App:** Unreal Engine 5.8 Editor
- **Bridge type:** a **port** — my own TCP server running inside the open Unreal Editor
- **Address:** `127.0.0.1:8765` (connections only from this computer)

This is not an installed third-party MCP server. `Content/Python/unreal_bridge_server.py` is the bridge: a background socket listener receives JSON commands, puts them in a queue, and a Slate editor-tick callback safely runs Unreal Python API calls on the editor thread.

The project includes Unreal's standard **Third Person** template. After the bridge builds the forest, press **Play**, then use **WASD** to move and **Space** to jump.

## How to run it

1. On Windows, install Unreal Engine 5.8.
2. Download `UnrealAgentBridge.zip` from this repository and unzip it. No separate Python or Codex CLI installation is needed.
3. Open `UnrealAgentBridge\\UnrealAgentBridge.uproject` from the unzipped folder. The project enables Unreal's built-in **Python Editor Script Plugin** and opens the forest map.
4. Wait until the Output Log says: `[AgentBridge] Listening on tcp://127.0.0.1:8765`.
5. Open PowerShell in the project folder and verify the bridge:

```powershell
powershell -ExecutionPolicy Bypass -File .\\run_bridge_client.ps1 health
```

6. Ask the agent for the forest in plain language. The agent runs this PowerShell command through the bridge:

```powershell
powershell -ExecutionPolicy Bypass -File .\\run_bridge_client.ps1 build_forest
```

Every run chooses a new seed, so the tree positions, trunk colours, and leaf colours change. To reproduce one forest, pass a seed:

```powershell
powershell -ExecutionPolicy Bypass -File .\\run_bridge_client.ps1 build_forest -Seed 482901
```

7. The editor opens a clean map at `/Game/AgentBridgeForest` and frames the forest from its entrance. Press **Play** and use **WASD** and **Space** to explore it.
8. Have the bridge inspect its own result:

```powershell
powershell -ExecutionPolicy Bypass -File .\\run_bridge_client.ps1 inspect_forest
powershell -ExecutionPolicy Bypass -File .\\run_bridge_client.ps1 screenshot
powershell -ExecutionPolicy Bypass -File .\\run_bridge_client.ps1 save
```

If the listener does not start automatically, open **Output Log** in Unreal and run:

```text
py "exec(open(r'Content/Python/init_unreal.py', encoding='utf-8').read())"
```

## What I asked for

> In Unreal Engine, create a stylized random forest I can explore as a Third Person character. Use black, brown, and white tree trunks; red, orange, and green leaves; warm sunlight; and grass that moves in the wind. Add a simple childlike course in a clearing, with low rectangular steps, a log bridge, a small slide, a seesaw, and a swing. Put an English sign that says “EXPLORE THE COURSE! WASD TO WALK - SPACE TO JUMP” at the entrance, then frame the editor view from there.

The agent sent `build_forest` to the local TCP port. Unreal created a clean forest map with 30 randomly placed low-poly trees, each using a black, brown, or white trunk and red, orange, or green foliage. It also created 80 animated grass blades, warm sunlight, a PlayerStart, and one readable `EXPLORE THE COURSE!` instruction sign on the entrance board. The low childlike obstacle course has five gentle steps, a log bridge, a tower and slide, a seesaw, a swing frame, and a finish block. `inspect_forest` checks the forest and course pieces before I press Play to explore it.

## Architecture

```text
Agent / PowerShell --> run_bridge_client.ps1 --> TCP 127.0.0.1:8765
                                                     |
                                                     v
                                   Unreal Python socket listener
                                                     |
                                              thread-safe queue
                                                     |
                                                     v
                                   Unreal editor tick + Unreal Python API
```

## Recording plan (under one minute)

1. Put Unreal and a terminal side by side; show the editor and the `[AgentBridge]` listener line.
2. Ask the agent the quoted request above.
3. Show `build_forest` returning its seed while the coloured trees, sunlight, instruction sign, and childlike course appear in Unreal.
4. Show `inspect_forest` passing, press **Play**, then walk briefly through the forest and climb the course with the Third Person character.
