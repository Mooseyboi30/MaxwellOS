# Build a MaxwellOS website

Copy [the template chip](starfall_website_template.txt), change its name and drawing commands, and upload it as a separate shared Starfall chip. Open **Desktop → Apps → Website Viewer** on MaxwellOS v35.6 or later. Aim at **Add one** or **Reset** and press **E**. **Enter** activates the first fully visible button in list order. This template targets Website Viewer. A physical Starfall screen needs its own client render hook.

## Remote hook contract

Register a Starfall `remote` hook with arguments `sender, user, protocol, action`.

| Protocol | Realm | Return values or behavior |
| --- | --- | --- |
| `starfall_website_discover_v1` | Server | Chip entity index, site name |
| `starfall_website_page_v1` | Server | Chip index, site name, title, fallback lines, mode, font, amounts, uptime, options |
| `starfall_website_screen_v1` | Server | Array of drawing commands. A client hook can optionally provide frames directly. |
| `starfall_website_button_v1` | Server | Handle the action ID passed as the fourth hook argument. The second argument is the requesting computer owner. |

For regular sites, return `false` for mode, amounts, and uptime. The font is `{family, size, weight}`. The viewer scales it to fit the mini screen. Keep fallback text to five lines of up to 42 printable ASCII characters each. Site names allow 32 characters, and titles allow 26.

## Graphics, animation, and scrolling

A frame contains 1–48 commands. Supported formats are:

```lua
{"rect", x, y, width, height, red, green, blue}
{"text", x, y, width, height, red, green, blue, text}
```

Keep RGB channels in 0–255 and coordinates inside the page. The page is 512 units wide and 512–1023 units tall. Keep each rectangle or text box at most 512 units high. Text is limited to 96 bytes per command and displays on one line. For animation, compute positions from `timer.curtime()` or from game state when the viewer requests a frame. Keep game state on the server if all viewers should see the same result.

A height above 512 enables scrolling. The viewer requests drawing frames only for the selected site. An unavailable or rejected feed can show the fallback title and text.

## Buttons

Publish up to 64 buttons using the ninth page return value:

```lua
local options = {512, buttons = {
    {96, 370, 150, 56, "Add one", "increment"},
    {266, 370, 150, 56, "Reset", "reset"}
}}
```

The first field is page height. Each button contains x, y, width, height, prompt text, and an optional action ID. Draw visible labels and backgrounds in the frame, and keep hitboxes inside the page and clear of each other. The 48-command graphics limit still applies. Leaving out the action ID displays the prompt locally. Existing sites can still use the flat single-button options format.

The action ID uses up to 40 ASCII letters, digits, underscores, or hyphens. Define a Lua function for each action in the website chip and call it from the server's `starfall_website_button_v1` handler. The template shows how to dispatch IDs through a Lua function table.

Handle only the action IDs your site defines. Validate the requesting player, permissions, and game state in your own handler before changing state or requesting in-game currency. Website Viewer supplies the display and input. The site owns its actions and data.

## Sound cues

On MaxwellOS v35.8 or later, return a cue ID from the server button hook after handling the action. Supported IDs are `click`, `success`, `error`, `splash`, and `chime`. The viewer plays the built-in sound once for the player who pressed the button, positioned at the linked screen. The pocket computer plays it locally. Return nothing for a silent action. See the template for examples.
