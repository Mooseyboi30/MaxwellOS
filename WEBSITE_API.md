# Build a MaxwellOS website

Copy [the template chip](starfall_website_template.txt), change its name and drawing commands, and upload it as a separate shared Starfall chip. Open **Desktop → Apps → Website Viewer** on MaxwellOS v34.6 or later. Aim at the example button and press **E**, or press **Enter** while it is fully visible. The shared counter should increase. This template targets Website Viewer. A physical Starfall screen needs its own client render hook.

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

Publish one interactive button per site using the ninth page return value:

```lua
{pageHeight, x, y, width, height, promptText, actionId}
```

The button bounds must fit inside the page. Draw its visible label and background in your frame. Use an action ID of up to 40 ASCII letters, digits, underscores, or hyphens. With an action ID, MaxwellOS forwards the click to your server hook. Without one, the viewer displays the prompt text locally. Omit the coordinates to publish a page with no button.

Handle only the action IDs your site defines. Validate the requesting player, permissions, and game state in your own handler before changing state or requesting in-game currency. Website Viewer supplies the display and input. The site owns its actions and data.
