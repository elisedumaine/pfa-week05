# which app, and which way (command, port, or MCP).

- **App:** Unreal Engine 5.8 Editor.
- **Bridge type:** a **port** — my own TCP server running inside the open Unreal Editor.
- **Address:** `127.0.0.1:8765`, so it accepts connections only from this computer.

This is not an installed third-party MCP server. `Content/Python/unreal_bridge_server.py` is the bridge: a background socket listener receives JSON commands, puts them in a queue, and a Slate editor-tick callback safely runs the corresponding Unreal Python API calls on the editor thread.

The project includes Unreal's standard **Third Person** template. That is what makes the result explorable: after the bridge builds the forest, press **Play**, then use **WASD** to move and **Space** to jump.

# How to run it. 
1. Install Unreal Engine 5.8 (or update `EngineAssociation` in `UnrealAgentBridge.uproject` to your compatible version).
2. Open `UnrealAgentBridge.uproject`. The project enables Unreal's built-in **Python Editor Script Plugin** and opens the Third Person map.
3. Wait until the Output Log says: `[AgentBridge] Listening on tcp://127.0.0.1:8765`.
4. In PowerShell at the project root, verify the bridge. The helper uses the Python that ships with Unreal, so a separate Python installation is not needed:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\run_bridge_client.ps1 health
   ```

5. Ask the agent for the forest in plain language. The agent runs this command through the bridge:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\run_bridge_client.ps1 build_forest
   ```

   Every run chooses a new seed, so the tree positions, trunk colours, and leaf colours change. To reproduce one forest, pass a seed:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\run_bridge_client.ps1 build_forest -Seed 482901
   ```

6. The editor opens a clean map at `/Game/AgentBridgeForest` and frames the forest from its entrance. Press **Play** and use **WASD** and **Space** to explore it. The agent has placed the Third Person PlayerStart at the edge of the clearing.
7. Have the bridge inspect its own result:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\run_bridge_client.ps1 inspect_forest
   powershell -ExecutionPolicy Bypass -File .\run_bridge_client.ps1 screenshot
   powershell -ExecutionPolicy Bypass -File .\run_bridge_client.ps1 save
   ```

If the listener does not start automatically, open **Output Log** in Unreal and run:

```text
py "exec(open(r'Content/Python/init_unreal.py', encoding='utf-8').read())"
```


# What you asked for

> In Unreal Engine, create a stylized random forest I can explore as a Third Person character. Use black, brown, and white tree trunks; red, orange, and green leaves; warm sunlight; and grass that moves in the wind. Add a simple childlike course in a clearing, with low rectangular steps, a log bridge, a small slide, a seesaw, and a swing. Put “FORET ALEATOIRE” at the entrance and frame the editor view from there.

The agent sent `build_forest` to the local TCP port. Unreal created a clean forest map with 30 randomly placed low-poly trees, each using a black, brown, or white trunk and red, orange, or green foliage. It also created 80 animated grass blades, a warm directional sun, skylight, sky atmosphere, light fog, a PlayerStart, the `FORET ALEATOIRE` 3D text, and a low childlike obstacle course: five gentle steps, a log bridge, a tower and slide, a seesaw, a swing frame, and a finish block. The response includes the random seed; `inspect_forest` checks the forest and course pieces before I press Play to explore it.

RECORDING : 

