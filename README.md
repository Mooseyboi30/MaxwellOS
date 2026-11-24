# MaxwellOS

A Starfall desktop for Garry's Mod with local apps, chatrooms, MaxwellNET websites, games, Paint and music.

Computer and chatroom server: **v50.4**. The standalone phone uses the same release version and includes the current shared desktop features.

## Install

1. Download the repository ZIP and extract it. Copy the chip files into `garrysmod/data/starfall/MaxwellOS` using their existing filenames.
2. Upload `starfall_chatroom_computer.txt` and link a Starfall screen. The computer bundles its helpers and needs no separate runtime or desktop-helper files.
3. Optionally wire a Wiremod Keyboard's key array and User outputs to `KeyboardKeys` and `KeyboardUser`. The `Power` input turns the computer off at 0.
4. Upload `starfall_chatroom_server.txt` to host a chatroom, or a website chip to publish a MaxwellNET site. Website Viewer works without joining a chatroom.
5. Alternatively, upload `maxwellos_phone.txt` for the standalone phone. Its shared helpers are bundled, so it needs no extra include files. Compass and TTS are phone-only features.

Grant Starfall permissions when prompted. Reupload a chip after updating its source; refreshing Desktop only reloads installed app files.

| File | Purpose |
| --- | --- |
| [Computer](starfall_chatroom_computer.txt) | Self-contained main desktop |
| [Chatroom server](starfall_chatroom_server.txt) | Rooms, channels, moderation, multiplayer services and saved server data |
| [Standalone phone](maxwellos_phone.txt) | Portable desktop with phone-specific additions |
| [Demo](starfall_computer_demo.txt) | Standalone offline demonstration |
| [App API](MOSAPPS.md) | Install and develop Lua apps |
| [Website API](WEBSITE_API.md) | Develop MaxwellNET sites |

## Desktop and controls

- **Ctrl:** Desktop / last app, using either Ctrl key or Wire keyboard codes 158/159.
- **Alt:** release keyboard focus. Press E on the screen to take control.
- **Click / double-click:** select / open a desktop shortcut. Aim and press E to open it directly.
- **Drag:** reorder desktop shortcuts. **Right-click:** pin/unpin a shortcut, up to five pins.
- **Arrow keys / Enter:** select and open apps. Page Up/Down scrolls the desktop.
- **R on Desktop:** manually refresh installed apps. Discovery runs once at startup; there are no periodic rescans.

Search filters app names. Start lists all programs. The desktop fills six columns with up to 42 shortcuts before scrolling.

The Desktop and Windows taskbar uses icons for open windows and ongoing background sessions: joined chatrooms, playing music and active multiplayer lobbies. An underline means running; the focused window is highlighted. Click an icon to restore/focus it. Inactive app history is not shown. Pinned shortcuts share their running app's icon and have no underline while closed. The OS button opens Start; R refreshes Desktop. Arrows scroll overflowing task icons or desktop columns.

## Apps and tools

| Tool | Controls and behavior |
| --- | --- |
| App Manager | Inspect source paths, dependencies, missing/changed files, companion status and errors; open, reload, disable/enable or pin apps. Disabling an app also unloads its automatic companion modules. |
| Notifications | Saved mentions, missed messages, game-lobby announcements and payment notices. Do Not Disturb suppresses notification popups and mention sounds. Up/Down scrolls, D toggles quiet mode, C clears. Stores the latest 50 entries. |
| Paint | 50 x 50 canvas with five save slots and chat sharing. Drag to draw/erase; Z/Y undo/redo, I line, Q rectangle, P brush, F fill. Shapes use two endpoints/corners. Keeps 20 edit snapshots. |
| Profile Transfer | Export/import name, theme, desktop layout/pins, favorites, wallpaper selection, Paint slots, notifications and local app saves. Uses `sf_filedata/maxwellos_profile_export.json`; leave the chatroom before importing. App source, wallpaper image files, server data and phone-specific settings remain separate. |
| Wire Dashboard | Four numeric `DashboardValue1`-`DashboardValue4` inputs with optional string `DashboardLabel1`-`DashboardLabel4` labels. Four numeric `Control1`-`Control4` outputs. Click to toggle 0/1, or select, type a number and press Enter. Owner-only controls reject changes while powered off. Reads values twice per second while open. |
| Windows | Up to two windows for Calculator, Chatrooms, Website Viewer, Files or installed apps. Click to focus, drag title bars, X closes, [] opens full-screen. Keyboard input goes to the focused window; unfocused installed apps pause. Games retain their normal full-screen views. |
| Files | Read, edit and create local .txt files up to 32768 bytes. F2 saves; Tab changes filename/editor focus. Unsaved edits are retained while switching apps. |
| Music and games | HL2/Episode music and the existing single/multiplayer game collection. Music and connected chatrooms retain their background behavior. |

Desktop settings save in `sf_filedata/maxwellos_desktop.json`; notifications save in `maxwellos_notifications.json`.

## Installable apps

Copy app files or entire package folders into `garrysmod/data/sf_filedata/mosapps`, then press R on Desktop. The legacy `data/starfall/MaxwellOS/mosapps` and `data/starfall/mosapps` folders also work. Icons are discovered from files; you do not need to place another chip.

- [Counter](mosapps/counter.txt): minimal single-file example.
- [Sprite Demo](mosapps/sprite_demo/app.txt): a folder package with a helper and automatically hosted companion module. Create/Hide controls a world sprite above the computer. R inside the app reloads its source and companion files. No extra processor is needed. The flat `sprite_demo.txt` is an optional alternate launcher with the same app ID.
- [Spinning Boxes](mosapps/spinning_boxes/app.txt): compares a built-in cube with geometry, color and speed loaded from `box_data.txt`. R reloads; Space pauses. Missing/invalid external data leaves the built-in cube running and shows an error for the other cube.

The [app guide](MOSAPPS.md) documents folder entries, dependencies, saved data, companions, input and window behavior. The optional standalone Sprite Demo screen uses `demo_chip.txt`, `runtime.txt` and `companion.txt` in its folder. `companion_chip.txt` demonstrates the older separately placed companion protocol.

## MaxwellNET and websites

MaxwellNET discovers Starfall chatroom and website chips on the same Garry's Mod server. Site discovery, graphics, animations, forms, buttons, subsites, sounds and payments use the in-game protocol. GitHub hosts chip downloads and update information; MaxwellNET is not a general internet browser.

Start with the [website template](starfall_website_template.txt), [storage helper](maxwellnet_storage.txt) and [Website API guide](WEBSITE_API.md). Sites remain independent of the computer/chatroom version numbers.

**Website Maker** builds sites with text, buttons, inputs and subsites. Add a route with + Subsite and keep a `home` route. A button's Go to property links to another route; its Message property provides feedback and can include `{input}`. Set the site name and .txt filename, then Export or press F2. Exports save in `sf_filedata`; copy the exported file into `data/starfall`, upload it as a website chip and open Website Viewer. The export bundles its own runtime.

## Wallpaper

Put PNG/JPG files in `garrysmod/data/sf_filedata/moswallpapers`. Open Settings > Wallpaper, rescan, select an image and Apply. The picker lists up to 64 images and saves the selection in `maxwellos_wallpaper.txt`. Images preserve their proportions with edges cropped to fill the desktop. Default restores the built-in background.

## Credits

Creator and contributor credits, with Steam profile links, are available in the Credits app. Thanks to [electr0n](https://steamcommunity.com/id/notanotherminge) for linking his OS with the server base.
