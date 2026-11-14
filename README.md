# MaxwellOS

A Starfall computer for Garry's Mod with a desktop launcher, chatrooms, a website viewer, and games.

## Setup
EXTRA: DO NOT COPY THE ENTIRE TEXT FILE INTO A NEW STARFALL CHIP INSIDE GMOD, THIS WILL MOST LIKEY BREAK IT AND MESS UP THE ENTIRE SYSTEM. Instead download the .zip, unzip it, then copy the chip you want into C:\Program Files (x86)\Steam\steamapps\common\GarrysMod\garrysmod\data\starfall, DO NOT DO ANY OTHER ROUTE THIS WILL BREAK IT BECAUSE YOU MIGHT MISS IMPORTANT CODE, and by COPYING it i mean copying the file itself into the directory stated, not any other route.

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
