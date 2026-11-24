# Build a MaxwellNET website

MaxwellOS v44.7 adds text fields, action responses, navigation, bookmarks, and subsites. Copy both `starfall_website_template.txt` and `maxwellnet_storage.txt` into your Starfall directory. Upload the template as a shared chip, then open Desktop → Website Viewer. Change the site's name and its storage key for your own project.

The example includes a Home form, a saved-note subsite, an About subsite with scrolling, pointer dragging, immediate sounds, and a delayed sound. The viewer renders the website inside its mini screen. A physical screen on the website chip needs its own client rendering and interaction hooks.

## Remote hook arguments

Register a shared chip's server `remote` hook with `sender, user, protocol, ...`. The `sender` is the calling processor and `user` is its owner. Validate actions and values on the website server.

| Protocol | Extra arguments | Return |
| --- | --- | --- |
| `starfall_website_discover_v1` | None | Chip entity index, site name |
| `starfall_website_page_v1` | capabilities, route | Chip index, site name, title, fallback lines, mode, font, amounts, uptime, options |
| `starfall_website_screen_v1` | capabilities, route | Frame, optional timed sound cue, optional event token |
| `starfall_website_button_v1` | action ID, form values, capabilities, route | Action response or legacy sound string |
| `starfall_website_input_v1` | field ID, text, capabilities, route | Action response |
| `starfall_website_pointer_v1` | x, y, held, right, capabilities, route | None |
| `starfall_website_capabilities_v1` | None | Calling this protocol on a v44.7 computer returns its capability table |

Legacy hooks can ignore the additional arguments. A missing route means `home`. For ordinary sites, return `false` for mode, amounts, and uptime. Supply a font as `{family, size, weight}`. The viewer scales it for the mini screen.

Site names support 32 bytes, titles 26, and fallback text five lines of 42 printable ASCII characters. Use a unique site name so links and favorites resolve consistently. Discovery displays up to 32 sites.

## Capabilities

The computer passes this table to the site's page, graphics, action, input, and pointer hooks.

```lua
{
    api = 2,
    buttons = true,
    pointer = true,
    textFields = true,
    actionResults = true,
    navigation = true,
    subsites = true,
    scrolling = true,
    sounds = true,
    storageHelper = true,
    maxButtons = 64,
    maxFields = 8,
    maxCommands = 384,
    maxHeight = 1023
}
```

Test individual flags before using a feature. Older viewers may pass no table. The storage flag means the helper is distributed with MaxwellOS. Each website must include and initialize that helper and check its readiness and save status.

Set `options.requires = {"textFields", "subsites"}` to declare required features. The viewer reports unsupported names. A site should still return readable fallback content.

## Graphics and scrolling

A frame contains 1–384 commands.

```lua
{"rect", x, y, width, height, red, green, blue}
{"text", x, y, width, height, red, green, blue, text}
```

Coordinates are page coordinates. Pages are 512 units wide and 512–1023 units tall. Keep each box inside the page bounds, with a maximum width and height of 512. RGB values must be 0–255. On v50.5 and newer, text supports 512 bytes per command and wraps across lines within the box height, including explicit newlines. Older viewers support only 96 bytes and one displayed line. Invalid frames produce an error message. A feed that stops responding shows a retry prompt after five seconds.

The first options value sets the page height. Heights above 512 enable Up/Down scrolling. The viewer fetches graphics for the selected site and route. Compute animation from the site's own clock or server state.

## Buttons, fields, and links

The ninth page return value contains the options table.

```lua
local options = {
    640,
    requires = {"textFields", "subsites"},
    buttons = {
        {28, 225, 216, 48, "Submit order", "submit_order"}
    },
    inputs = {
        {28, 155, 456, 48, "Your order", "order", 80, ""}
    },
    links = {
        {28, 300, 216, 44, "Order history", "", "orders"},
        {268, 300, 216, 44, "Other site", "Maxwell Cola", "home"}
    }
}
```

| Item | Fields | Limit |
| --- | --- | --- |
| Button | x, y, width, height, prompt, action ID | 64 |
| Input | x, y, width, height, placeholder, field ID, max bytes, initial text | 8 |
| Link | x, y, width, height, label, destination site name, destination route | 16 |

Keep hitboxes within the page and avoid overlapping them. Action IDs and field IDs support 1–40 ASCII letters, digits, underscores, or hyphens. Field IDs must be unique within a page. Text fields accept up to 160 bytes each. The viewer draws fields and links. Sites draw button backgrounds and labels in their graphics frame.

Click a field to edit it. Tab cycles fields. Wiremod users can press I to focus the first field, then Enter to submit. Escape releases field focus. Physical and Wiremod keyboards use the same input handler. Buttons receive a table of all current field values, indexed by field ID. Enter in a field calls the input hook with that field's value.

A legacy button with no action ID displays its prompt locally. The earlier flat single-button options format remains supported.

## Action responses

Buttons and submitted fields can return a response table.

```lua
return {
    ok = true,
    message = "Your order was received.",
    sound = "success",
    route = "orders"
}
```

Supported keys are `ok`, `message` (160 bytes), `sound`, `navigate` (a destination site name), and `route`. Returning only `route` opens that subsite on the current chip. Combining `navigate` and `route` opens another discovered site. The response is delivered to the requesting viewer. Return nothing for a silent action, or a legacy sound ID string for an immediate sound.

The website owns its data, actions, and payment requests. Validate players, form contents, permissions, prices, and game state before changing data or moving in-game currency. The viewer bounds and cleans input, and accepts actions only for its selected website.

## Subsites and navigation

A single chip can serve different options and frames according to the route argument. Routes support up to 64 letters, digits, underscores, hyphens, or slashes. The default is `home`. Routes are logical names handled by your script.

```lua
if protocol == "starfall_website_page_v1" then
    local capabilities, route = first, second or "home"
    return chip():entIndex(), NAME, NAME, {"Example site"},
        false, FONT, false, false, optionsFor(route)
elseif protocol == "starfall_website_screen_v1" then
    local capabilities, route = first, second or "home"
    return frameFor(user, route)
end
```

The Back button or H returns to the previous site/route. Save or B toggles a favorite for the current site and route. Favorites or K opens the saved list. Up/Down selects a favorite and Enter opens it. Click a favorite to open it. Retry or R reloads the current route.

Bookmarks save to the computer user's local `maxwellnet_bookmarks.json` through the Starfall file API. They require file access and store the site name and route. The website chip must be loaded and discoverable when opening a favorite. Back history lasts for the current chip session and holds up to 32 entries. Favorites hold up to 32 entries.

## Pointer dragging and scratch coatings

Pointer coordinates include the current page scroll offset. The server receives pointer events while a mouse or Use button is held, at roughly ten events per second.

```lua
elseif protocol == "starfall_website_pointer_v1" then
    local x, y, held, right = first, second, third, fourth
    local capabilities, route = fifth, sixth or "home"
    -- Record movement through your scratch areas here.
end
```

Use per-player state for private scratch areas. Track movement between samples and reveal only the touched region or cells. The template's dragging strip demonstrates progress from horizontal movement. Release events set both held flags to false when the viewer can report the release.

## Immediate and timed sounds

Immediate sound IDs are `click`, `success`, `error`, `splash`, and `chime`. They play at the linked computer screen, or locally in the pocket viewer.

For delayed server events, return a frame followed by `drop` or `win` and a unique token.

```lua
return frame, "win", event.token
```

Tokens support 48 bytes. Keep a token stable across polls for the same event and keep the cue available briefly so polling can pick it up. The viewer plays each token once while that site is selected. Keep private sound events indexed by the requesting player. Timed cues use the server frame relay, so omit the client frame hook when using them.

## Persistent storage helper

Include the helper file and initialize it in both realms.

```lua
--@include maxwellnet_storage.txt
local storage = require("maxwellnet_storage.txt")("my_unique_site")
```

Keep the storage key stable through uploads. It supports 1–40 ASCII letters, digits, underscores, or hyphens. Different sites should use different keys. Saved data lives in the website owner's local Starfall file directory under `maxwellnet_store_my_unique_site.json`.

| Member | Behavior |
| --- | --- |
| `storage.ready` | True after a successful owner-side load |
| `storage.status` | Loading, saving, saved, or an error message |
| `storage.get(key, default)` | Server-side read, available once ready |
| `storage.set(key, value)` | Server-side update, returns success and optional error |
| `storage.delete(key)` | Server-side deletion using the same save queue |

Data must be JSON-compatible and the full save must fit within 16000 bytes. Values returned by get are copies. Updates queue a write and the owner acknowledges it after read-back verification. `set` returning true means queued. Wait for `status == "Saved"` before reuploading or removing the website chip. Pending writes retry every two seconds. Initial load errors leave the store unready so defaults cannot overwrite unread data.

The website owner must be connected and grant file permissions. The local file belongs to that owner's computer. This helper suits notes, settings, and ordinary site data. Currency transfers require the server payment API and separate settlement logic.

### Discovery announcements

Generated sites also announce their name and chip ID to computers every three seconds. v46.1 computers and updated standalone phones keep these announcements for ten seconds as a fallback when direct discovery misses a site. Website graphics are requested when selected.

A server-side site can publish the same metadata with:

```lua
hook.runRemote(nil, "starfall_website_register_v1", chip():entIndex(), NAME)
```

Generated exports now contain native Lua page tables so discovery does not depend on JSON decoding during startup.
