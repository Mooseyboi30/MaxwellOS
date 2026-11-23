# Example apps

Copy these apps into `garrysmod/data/sf_filedata/mosapps`, then press R on Desktop. Copy package folders intact. The computer/phone must already have the current mosapps loader.

| Example | Files to install | What it tests |
| --- | --- | --- |
| Counter | `counter.txt` | Single-file app, input and saved data |
| Sprite Demo | Entire `sprite_demo/` folder | Relative `ui.txt` helper, automatically loaded `companion.txt`, world sprite and request replies |
| Spinning Boxes | Entire `spinning_boxes/` folder | Built-in animation beside geometry/color/speed loaded from `box_data.txt` |

`sprite_demo.txt` is an optional flat launcher for Sprite Demo. It uses the same app ID and yields to the folder package, so installing both gives one icon. It still needs `sprite_demo/companion.txt`.

The Sprite Demo folder also contains optional standalone-chip examples:

- `demo_chip.txt` with `runtime.txt` and `companion.txt`: upload a button-screen chip and link a screen.
- `companion_chip.txt`: older separately placed companion protocol example. It is not needed by the installable app.

Do not upload `app.txt` as a separate Starfall chip. MaxwellOS loads app entries from the local folder. R inside Sprite Demo or Spinning Boxes reloads their files. See [the app API](../MOSAPPS.md) for descriptor and dependency rules.
