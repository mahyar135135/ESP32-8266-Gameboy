# ESP8266 OLED Console - v1.2.0 "Duel"

Eighteen games plus a settings menu for an **ESP8266 (NodeMCU)** with a **0.96" 128x64 SSD1306 I2C OLED** and **5 buttons**.
Built on your original `sketch_oct27a.ino` (Dino Run, Flappy, Pong, Breakout, EEPROM high scores, same SELECT-hold-to-leave idea).

One-player games: Dino Run, Flappy, Pong, Breakout, Snake, Blocks, Invaders, Mines, Connect 4, TicTacToe (vs CPU), Muncher, Tanks (vs CPU).
**Two-player games (new in 1.2, each is its own app in the menu):** Pong 2P, Connect 2P, TicTac 2P, Tanks 2P, Cycles 2P, Reversi 2P.

## Verification status (please read)
* Versions 1.0 and 1.1 were compiled and you confirmed they work on hardware.
* **Version 1.2 (six 2-player games, new sounds, new settings, bigger About page, storage layout 3) was written
  carefully but has NOT been compiled or run.** If the compiler reports an error, please send me the message.
* Version number, codename and the number of games now live in `app_info.h`, so **your own `config.h`
  (pin wiring, e.g. RX/TX buttons) can stay as it is** - this update does not need anything from it.

## Hardware / pins (all buttons go between the GPIO and GND; internal pull-ups are used)

| Function | NodeMCU pin | GPIO | Notes |
|---|---|---|---|
| OLED SDA | D2 | 4 | I2C, address 0x3C (change in `config.h`) |
| OLED SCL | D1 | 5 | |
| UP | D5 | 14 | |
| DOWN | D6 | 12 | |
| SELECT | D7 | 13 | |
| LEFT | D3 | 0 | boot-strap pin: don't hold while pressing reset |
| RIGHT | D4 | 2 | boot-strap pin, shares the on-board LED (it may glow faintly) |
| Buzzer (optional) | D8 | 15 | passive buzzer to GND. **Needs extra hardware**; without it the Sound setting does nothing |

OLED VCC = 3.3 V, GND = GND. UP/DOWN/SELECT keep the pins from your original sketch.
Note: your original called `Wire.begin(5,4)` (SDA=GPIO5), but `display.begin()` then re-initialised I2C with the default pins
(SDA=GPIO4, SCL=GPIO5), so D2=SDA / D1=SCL is almost certainly what your display is wired to. This project uses that explicitly.
If your wiring differs, edit `PIN_SDA` / `PIN_SCL` in `config.h`.

## Libraries (Arduino IDE -> Library Manager)
* **Adafruit GFX Library**
* **Adafruit SSD1306**
* **Adafruit BusIO** (installed automatically as a dependency)

EEPROM and Wire come with the ESP8266 core. Install the core via Boards Manager ("esp8266 by ESP8266 Community").

## Install / upload
1. Unzip. Keep the folder name `ESP8266_OLED_Console` (it must match the `.ino` name).
2. Open `ESP8266_OLED_Console.ino` in Arduino IDE.
3. Tools -> Board -> **NodeMCU 1.0 (ESP-12E Module)**, Flash size 4MB, upload speed 115200 or 921600.
4. Select the port and press Upload. Serial monitor (115200) only prints an error if the OLED isn't found.

## Controls (same everywhere)
| Action | Button |
|---|---|
| Move / navigate | UP, DOWN, LEFT, RIGHT (hold = auto-repeat in menus) |
| Confirm / start / action | SELECT (short press) |
| Pause menu (Resume / Restart / Quit) | **hold SELECT** (~0.6 s) while playing |
| Back / leave | hold SELECT, or LEFT on title, game-over and confirm screens |
| Settings: change value | LEFT / RIGHT (or SELECT to toggle/cycle) |

Game-over screen ignores buttons for 0.7 s to prevent accidental restarts.
The board games (Mines, Connect 4, TicTacToe) first show the final board; press SELECT to see the score panel.

| Game | Controls | Notes |
|---|---|---|
| Dino Run | UP/SELECT jump, DOWN duck (fast-fall in air) | birds appear later; speed ramps up |
| Flappy | UP/SELECT flap | score = pipes passed |
| Pong | UP/DOWN move, SELECT serve | you are the right paddle; score = points won before CPU reaches 10 |
| Breakout | LEFT/RIGHT, SELECT/UP launch | redesigned to the classic bottom paddle (the original was rotated because only UP/DOWN existed); lives, levels, 2-hit bricks |
| Snake | D-pad | Easy: walls wrap; Normal/Hard: walls kill. Bonus food every 5 apples |
| Blocks | LEFT/RIGHT move, UP rotate, DOWN soft drop, SELECT hard drop | ghost piece, next piece, 7-bag, levels |
| Invaders | LEFT/RIGHT, SELECT/UP fire | bunkers, UFO, waves, 3 lives |
| Mines | D-pad move, SELECT dig, **double-tap SELECT = flag** | 16x7 board, first dig is safe, SELECT on a satisfied number opens its neighbours, time bonus |
| Connect 4 | LEFT/RIGHT choose column, SELECT drop | vs CPU (search depth grows with difficulty); win a round = points and continue, lose a round = game over |
| TicTacToe | D-pad move, SELECT place | win 3 pts, draw 1 pt, lose = game over; Hard CPU never errs |
| Muncher | D-pad steer | 3 ghosts with different behaviour, power pellets, wrap-around tunnel, new mazes get faster |
| Tanks | UP/DOWN angle, LEFT/RIGHT power, SELECT fire | wind, destructible terrain, CPU gets more accurate every round |

## Two-player games (one console, two people)
| Game | Player 1 | Player 2 | Rules |
|---|---|---|---|
| Pong 2P | UP / DOWN | LEFT = up, RIGHT = down | SELECT serves, first to 7. High score = longest rally. |
| Connect 2P | LEFT/RIGHT, SELECT drop | same buttons, turn by turn | Solid discs vs hollow discs. Starter alternates. |
| TicTac 2P | D-pad + SELECT | same buttons, turn by turn | X vs O. Starter alternates. |
| Tanks 2P | UP/DOWN angle, LEFT/RIGHT power, SELECT fire | same, turn by turn | Wind, destructible terrain. |
| Cycles 2P | UP = turn left, DOWN = turn right | LEFT = turn left, RIGHT = turn right | Light-cycle duel, first to 3 rounds. High score = longest round (s). |
| Reversi 2P | D-pad + SELECT | same buttons, turn by turn | Dots show legal moves, automatic pass. High score = biggest winning margin. |

Versus games show a session win counter (kept until the console is restarted). Games without a meaningful record
(Connect 2P, TicTac 2P, Tanks 2P) have no high score.

## Settings (saved to EEPROM)
Brightness (16 steps), Invert, Flip screen (rotates 180 degrees and swaps the D-pad),
**Buzzer sound** (Off / Menu only / All), **Buzzer pitch** (Low / Normal / High, useful because buzzers differ),
Difficulty, Button repeat speed, Animation speed (UI only), **Screen sleep** (Off / 30 s / 1 min / 5 min: the panel switches off
when idle, any button wakes it and that press is ignored; it never sleeps during a running round),
**Hold time** (how long you hold SELECT for pause/back: 0.4 / 0.6 / 0.9 s), **Splash screen** on/off,
**Test sound**, Reset scores and Restore defaults (both with confirmation), About / System.
Buzzer settings need the optional buzzer (extra hardware); everything else works on the base console.

### Sounds
About 30 effects (menu tick, confirm/back, pause/resume, jump, shoot, cannon, explosion, flag, reveal, disc drop, flip,
munch, ghost, fanfare for a new high score, ...). Strong sounds are not cut off by weak ones. Nothing is played at power-up.

### About / System page (scroll with UP/DOWN)
Software (version, build date/time, core and SDK versions), hardware (chip ID, CPU, flash size/speed, boot reason),
memory (free heap, largest block, fragmentation, sketch size, free program space), display and pins, current settings,
storage (layout version, bytes used, scores saved, flash writes this boot, migration status) and session (uptime, rounds played).

## Storage
128-byte EEPROM window (room for 24 games): magic number + layout version + length + CRC-16 + settings + one high score per game.
Invalid or old data is replaced by defaults. Flash is written only when data really changed, and only at safe moments
(game over, leaving a menu), never during gameplay. Data saved by versions 1.0 and 1.1 (settings and high scores) is migrated automatically; the old on/off sound setting becomes Off / All.
High scores from your very first original sketch are not migrated (layout and scoring changed).

## Code layout
`ESP8266_OLED_Console.ino` (setup/loop only), `app.*` (state machine), `config.h`, `input.*`, `gfx.*`, `sound.*`,
`storage.*`, `bitmaps.*`, `game.*` (base class + title/pause/game-over runner), `games.*` (registry),
`game_*.cpp` (18 games), `app_info.h`, `main_menu.*`, `settings_menu.*`.
Games run on a fixed 33 ms step with `millis()`; there are no blocking delays in the loop. WiFi is switched off.

## Troubleshooting
* Blank screen: check SDA/SCL and `OLED_ADDR` (try 0x3D); lower `OLED_I2C_HZ` to 100000.
* Upload fails: unplug LEFT/RIGHT buttons' presses (GPIO0/GPIO2) during reset.
* Too fast/slow: tweak constants at the top of each `game_*.cpp`.
