# Installable MaxwellOS apps

MaxwellOS v50.3 and the updated standalone phone automatically discover app modules from local files. Place apps in the first folder below. No app chip needs to be placed for its desktop icon to appear.

- `garrysmod/data/sf_filedata/mosapps`
- `garrysmod/data/starfall/MaxwellOS/mosapps`
- `garrysmod/data/starfall/mosapps`

Save a module as .txt or .lua. The desktop scans once at startup. After adding or updating files, return to Desktop and press R or click the taskbar R button to reload its app list. There are no periodic rescans. The example [Counter](mosapps/counter.txt) can be copied straight into mosapps. App packages can be copied as complete subfolders. Identical copies across supported locations share one icon; different apps with duplicate IDs are reported.

The desktop has one icon list for installed apps, built-in apps, and games. Arrow keys select an icon, Enter opens it, Page Up/Down scrolls, and Tab opens Start. Double-click or aim and press E to launch an icon. A single click selects it. Start lists all programs. Ctrl switches between the desktop and the last app. Both Ctrl keys work, including Wire key codes 158/159; Alt releases keyboard focus.

## App format

Each file returns an app descriptor. The loader passes its context as the chunk argument and as the first argument of every callback.

```lua
local context = ...

return {
    id = "my_app",
    name = "My App",
    description = "A small installable app",
    needsRoom = false,

    open = function(api)
        api.state.count = api.state.count or 0
    end,

    draw = function(api)
        local theme = api.theme()
        render.setColor(theme.background)
        render.drawRect(0, 0, api.width, api.height)
        api.label(20, 20, "Count: " .. api.state.count)
    end,

    key = function(api, key, enter, character)
        if enter then api.state.count = api.state.count + 1 end
    end,

    click = function(api, x, y, right)
        api.state.count = api.state.count + (right and -1 or 1)
    end,

    update = function(api, dt, now)
        -- Optional per-frame app logic.
    end,

    close = function(api)
        -- Stop any sounds and release resources you created.
    end,

    pause = function(api)
        -- Optional cleanup while the computer is off or booting.
    end,

    resume = function(api)
        -- Optional restart after power returns.
    end
}
```

Only id, name, and draw are required. IDs support 1–32 lowercase ASCII letters, numbers, underscores, or hyphens. Names display up to 32 bytes. IDs must be unique. Duplicate IDs are reported, with the first file taking precedence. The sf_filedata/mosapps folder is scanned first, followed by Starfall/MaxwellOS/mosapps and root starfall/mosapps, each in filename order.

The loader queues up to 96 matching file/folder candidates across the supported locations, processing one per desktop frame. A module can contain up to 65536 bytes. Existing standalone Starfall chips need to be adapted to this descriptor format.

## Lifecycle and controls

open runs when the app is selected. draw, key, click, and update run while the app is selected and the computer is powered on. close runs when the user switches away. The app's state table stays in memory when returning to the desktop and reopening it.

Updates receive elapsed seconds as dt, capped at 0.05, plus the current timer value. The host invokes updates at up to 60 Hz. Returning to the desktop suspends the custom app. Powering off invokes pause once, and resuming invokes resume once.

Use callbacks for app work. Apps share the Starfall instance's CPU and memory limits. Stop sounds and release resources in close and pause. Ordinary callback errors stop that app and display an error panel. Rescan retries failed apps. Syntax and descriptor errors are reported on the desktop while other apps continue loading.

The key callback receives the same codes as the computer's Wiremod keyboard handler. Arrows are 17–20, printable characters are ASCII, Backspace is 8 or 127, and enter is supplied separately as a boolean. The character argument contains printable text when available. F1 remains the desktop shortcut.

The click callback receives coordinates relative to the app window, with right=true for a physical screen right click. E and left click supply right=false.

## App context

| Member | Behavior |
| --- | --- |
| api.id | Validated descriptor ID, available to callbacks |
| api.width, api.height | 480 by 380 content area |
| api.state | Private per-instance Lua table retained across openings |
| api.label(x,y,text,color,font) | Draw local text with optional color and font |
| api.wrap(text,width,font) | Return wrapped lines |
| api.theme() | Current background, panel, accent, text, and muted colors |
| api.home() | Return to desktop on the next update |
| api.room() | Current joined chatroom name, or an empty string |
| api.read(key,default) | Read a saved string for this app |
| api.write(key,value) | Save a string up to 32768 bytes |
| api.isKeyDown(KEY.W) | Held-key state from the physical or active Wiremod keyboard |

The render adapter offsets drawRect, drawRectOutline, drawText, drawCircle, drawCircleOutline, and drawLine into the app window. Use coordinates from 0,0 to api.width,api.height and keep drawings inside that area. Font and color setters use ordinary Starfall rendering APIs.

The module environment includes assert, error, pcall, pairs, ipairs, next, type, tonumber, tostring, select, unpack, setmetatable, getmetatable, math, string, table, json, Color, Vector, Angle, render, sound, KEY, MOUSE, timer.curtime, and print. Use the context for persistent state and file storage.

The module executes once to return its descriptor. Initialize saved state in open, after its ID has been validated. Per-app files use `mosapp_<id>_<key>.txt` under `garrysmod/data/sf_filedata`. Storage keys support 1–40 ASCII letters, digits, underscores, or hyphens. File access requires Starfall permissions.

## Updating and removing apps

Replace the module file, return to Desktop, and press R to load the change. Rescans preserve the instance when its source is unchanged. Updated modules get a fresh instance, so save important state in close or through an explicit Save action.

Delete the module file to remove its icon on the next desktop scan. Saved data remains in sf_filedata. Settings and Credits remain adjacent at the end of the program list.


## Folder packages and companion chips

The PC loader and updated standalone phone also support one folder per app.

```text
mosapps/
  sprite_demo/
    app.txt
    ui.txt
    companion.txt
    runtime.txt
    companion_chip.txt
    demo_chip.txt
```

The entry point is app.txt, app.lua, main.txt, or main.lua, in that priority order. Entry names are matched without case sensitivity while preserving their actual filename. DATA discovery merges the regular and game-file directory listings; each folder is processed on a later frame. If the package listing is incomplete, the loader tries those four standard filenames directly. A package with a load error keeps an error icon you can open to read the cause. Only that entry gets a desktop icon. Other files remain available as helper modules, automatically hosted companion modules, or optional standalone chips. Folder packages and single-file apps can coexist.

Use `require("ui.txt")` or `api.require("ui.txt")` to load a helper relative to the entry point's folder. A helper can return a table, function, or another Lua value. Modules use the same app environment, are cached per instance, and can require other modules within the same package. Circular dependencies and paths outside the package are rejected. Modules support .txt and .lua, up to 16 loaded helpers of 65536 bytes each. Changing a loaded helper causes the app to reload on the next desktop scan.

Use .txt for modules in sf_filedata, including Lua helper source. The bundled example uses ui.txt.

### Automatically hosted companions

Declare `companion = "companion.txt"` on the app descriptor, or `companions = {"first.txt", "second.txt"}` for up to four modules. Opening the app streams these files to the computer's existing server realm. Their client code and published state are then sent to viewers. No extra processors are created or placed. The PC includes mos_companion_runtime.txt and mos_desktop_tools.txt; keep both beside the PC source when uploading. The standalone phone bundles this runtime and needs only maxwellos_phone.txt.

A companion returns a shared descriptor:

```lua
local service = {role="example", name="Example Service"}
if SERVER then
    function service.init(api)
        api.state.count = api.state.count or 0
    end
    function service.request(api, action, payload)
        if action ~= "increment" then return {ok=false,error="Unknown action"} end
        api.state.count = api.state.count + 1
        api.publish()
        return {ok=true,count=api.state.count}
    end
else
    function service.update(api, dt, now)
        -- api.state is the most recently published server state.
    end
    function service.close(api)
        -- Remove any holograms, meshes and materials created by this module.
    end
end
return service
```

Callbacks are init(api), request(api,action,payload) on the server, update(api,dt,now), and close(api). api.localState is realm-local; api.position() returns the host processor's position. Server api.publish() broadcasts api.state as JSON (maximum 4096 bytes). The environment supplies standard Lua helpers, Color/Vector/Angle, chip/owner/player/eyePos, hasPermission, hologram/mesh/material/prop/find/entity, and timer.curtime. Engine libraries are available only where Starfall supplies them and still require their normal permissions. Modules share the host's quota and permissions; install trusted source.

Each source is limited to 65536 bytes, with at most sixteen hosted modules per processor. Discovery returns hosted targets as `local:role`; pass the returned id directly to api.callCompanion. The app's open callback waits for installation acknowledgment. Transfers time out with an app error. Services remain installed until the processor is reuploaded or removed; their updates can maintain world objects while their app window is closed. Replacing module source preserves published state and calls close on the prior module.

The legacy companion_chip.txt demonstrates the optional separately placed service protocol below. It is not needed for the automatic Sprite Demo.

### Companion API

```lua
open = function(api)
    local ticket, err = api.findCompanions("sprite")
end,

reply = function(api, ticket, ok, result, err)
    if not ok then
        api.state.status = err
        return
    end
    -- Discovery also updates api.companions.
end
```

`api.findCompanions(role)` asynchronously finds hosted modules and uploaded chips that support this app's descriptor ID. Omit role to find all roles. `api.companions` contains entries with id, role, and name.

`api.callCompanion(chipId, action, payload)` sends a request through the computer's server realm. Both calls return a numeric ticket, or nil and an immediate error. Keep the ticket to match its reply. Requests are limited to one every 0.2 seconds and 16 outstanding tickets. Payloads must fit in 2048 JSON bytes. Replies support up to 6000 JSON bytes and time out after six seconds.

The reply callback is `reply(api, ticket, ok, result, err)`. Transport success is separate from the companion's application-level result. A service may return `{ok=false,error="..."}` as its result. `api.lastReply` also stores the ticket, kind, ok, and value. Responses arriving while the app is closed are queued for its next opening, without running inactive app callbacks.

A server-side companion implements these protocols.

| Protocol | Arguments after sender and user | Return |
| --- | --- | --- |
| maxwellos_app_companion_discover_v1 | app ID | This chip's index, role, display name |
| maxwellos_app_companion_call_v1 | app ID, action, payload | JSON-compatible response |

Check the app ID and validate the caller, action, and data before changing world state. The requesting user is the computer chip's owner. The sample service accepts commands from chips owned by its owner.

### Sprite demo setup

1. Copy the entire sprite_demo folder into data/sf_filedata/mosapps.
The bundled mosapps/sprite_demo.txt is an alternate flat launcher for installations whose directory listing omits app folders. Keep sprite_demo/companion.txt with it. The launcher and folder package share one desktop icon. A descriptor with fallback=true yields to an ordinary package with the same ID.

2. Reupload the updated PC with mos_companion_runtime.txt beside its source, or upload the single-file maxwellos_phone.txt. Return to Desktop and press R.
3. Open Sprite Demo. Its companion.txt loads automatically. Press Create sprite and grant hologram/material permissions when prompted.
4. A camera-facing glow appears above the computer processor. Hide sprite hides it. Press R inside Sprite Demo to reread the app, helpers and companion files and retry the companion connection. This restarts app-local state without reuploading the computer. Existing server companion state is retained when source is unchanged.

For a standalone screen demonstration, upload demo_chip.txt with runtime.txt and companion.txt in its folder, then link a Starfall screen. The owner can press E on Create sprite or Hide sprite. This chip also loads its companion module automatically.

### Input rollover

Physical keyboard repeat tracks each held key independently. Releasing one key leaves other held keys active. The Wiremod path batches simultaneous repeated keys and publishes a held-key snapshot, so `api.isKeyDown(KEY.W)` and `api.isKeyDown(KEY.SPACE)` can both be true. Native controls use the focused physical keyboard, and Wiremod controls use the active keyboard's Keys array.

App callbacks can call `api.reload()` to request a fresh app instance from disk on the next update. Companion file caches and outstanding requests for that instance are reset; code is uploaded again and rediscovered. On an app error or companion-loading screen, R also retries loading.


## Spinning Boxes file-dependency test

Install the complete mosapps/spinning_boxes folder in data/sf_filedata/mosapps, then press R on Desktop. Spinning Boxes renders a built-in cube beside a cube whose vertices, edges, RGB color and rotation speed come from box_data.txt using the package require API. No companion processor or chatroom server is needed.

Inside the app, R or Reload files rereads both source files. Space/Enter or Pause stops and resumes both animations. Edit box_data.txt to test the external cube independently: color uses RGB values from 0 to 255; speed is radians per second from -3 to 3; vertices use coordinates from -1 to 1; edges reference vertex indices. The external file supports up to sixteen vertices and thirty-two edges. If it is missing or invalid, its panel displays an error while the built-in cube keeps running. Restore the file and reload to recover. Updates stop while the app is closed.


## Desktop management and windows

App Manager lists built-in and installed apps, exposes package/dependency failures, and provides reload, disable/enable and pin controls. Disabled apps retain their saves and source files; their hosted companions are unloaded. Re-enable and open the app to install those modules again. Missing module files appear in the manager's dependency status.

Installed apps can also open from the Windows workspace App picker. The existing draw/click/key/update API is unchanged: the window renders the app's normal screen into a smaller viewport and maps clicks back to app coordinates. Only the focused installed app runs update and receives keyboard input; unfocused installed windows retain their last rendered frame. Full-screen app behavior and manual desktop rescanning remain available.

Profile Transfer includes local mosapp_<id>_<key>.txt saves. It does not distribute executable app modules. Desktop icon order, taskbar pins and disabled state are persisted separately from app-specific state.

The Desktop and Windows taskbar shows app icons for open windows and ongoing background sessions (joined chatrooms, playing music and active multiplayer lobbies). A line below an icon means the app is running; the focused app is highlighted. Clicking an icon restores/focuses its window or opens the active session. Previously used inactive apps are not kept as taskbar history. Pinned shortcuts remain visible without a running line while closed; a pinned running app shares one icon. Right-click an icon to pin/unpin it. The OS button opens Start, R refreshes Desktop, and arrows scroll overflowing task icons or desktop columns.
