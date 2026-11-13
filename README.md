# MaxwellOS

A Starfall computer for Garry's Mod with a desktop launcher, chatrooms, a website viewer, and games.

## Setup

1. Upload the computer chip with StarfallEx and link it to a Starfall screen.
2. Wire a Wiremod Keyboard's Keys and User outputs to KeyboardKeys and KeyboardUser. The optional Power input turns the display off at 0.
3. Upload the chatroom server chip to host chat, or a website chip to publish a site. Open Chatrooms or Website Viewer from the desktop.
4. Use the standalone demo chip for Calculator and Minesweeper without a chatroom or website server.

   Congrats to https://steamcommunity.com/id/notanotherminge/ for making his OS link with my server base

## MaxwellNET

MaxwellNET is MaxwellOS's in-game network for discovering Starfall chatrooms and websites on the same Garry's Mod server. Open **Chatrooms** to join a room hosted by a MaxwellOS server chip, or open **Website Viewer** to browse standalone website chips. Website Viewer works without joining a chatroom.

MaxwellNET sites publish their own text, graphics, animations, and button actions through the Website Viewer protocol. A site can use that protocol to run its own games or interactive pages. Discovery and site traffic happen inside Garry's Mod. GitHub hosts the chip downloads and supplies version information for the update checker.

## Website development

Start with the [website template](starfall_website_template.txt) and the [Website Viewer protocol guide](WEBSITE_API.md).
