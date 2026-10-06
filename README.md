# ESP8266 OLED Console

Twelve games plus a settings menu for an **ESP8266 (NodeMCU)** with a **0.96" 128x64 SSD1306 I2C OLED** and **5 buttons**.
Built on your original `sketch_oct27a.ino` (Dino Run, Flappy, Pong, Breakout, EEPROM high scores, same SELECT-hold-to-leave idea).

Games: Dino Run, Flappy, Pong, Breakout, Snake, Blocks (falling-block puzzle), Invaders, and (v1.1) **Mines, Connect 4, TicTacToe, Muncher (maze chase), Tanks (artillery duel)**.

## Verification status (please read)
* Version 1.0 (the first seven games) was compiled with arduino-cli for `esp8266:esp8266:nodemcuv2`
  (core 3.1.2, Adafruit GFX 1.12.6, SSD1306 2.5.17, BusIO 1.17.4) and you reported it works on hardware.
* **Version 1.1 changes (the five new games, storage migration, game-over review stage, Dino and Pong fixes)
  were written carefully but have NOT been compiled or run.** If the compiler reports an error, please send me the message.

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

## Settings (saved to EEPROM)
Brightness (16 steps), Invert, Flip screen (rotates 180 degrees and swaps the D-pad so it still feels right when the console is turned around),
Sound (buzzer, needs extra hardware), Difficulty, Button repeat speed, Animation speed (UI/menu animations only; game physics stays constant),
Reset scores (with confirmation), Restore defaults (with confirmation), About / System info.

## Storage
64-byte EEPROM window: magic number + layout version + length + CRC-16 + settings + one high score per game.
Invalid or old data is replaced by defaults. Flash is written only when data really changed, and only at safe moments
(game over, leaving a menu), never during gameplay. Data saved by version 1.0 (settings and the first 7 high scores) is migrated automatically.
High scores from your very first original sketch are not migrated (layout and scoring changed).

## Code layout
`ESP8266_OLED_Console.ino` (setup/loop only), `app.*` (state machine), `config.h`, `input.*`, `gfx.*`, `sound.*`,
`storage.*`, `bitmaps.*`, `game.*` (base class + title/pause/game-over runner), `games.*` (registry),
`game_dino/flappy/pong/breakout/snake/blocks/invaders/mines/connect4/tictactoe/muncher/tanks.cpp`, `main_menu.*`, `settings_menu.*`.
Games run on a fixed 33 ms step with `millis()`; there are no blocking delays in the loop. WiFi is switched off.

## Troubleshooting
* Blank screen: check SDA/SCL and `OLED_ADDR` (try 0x3D); lower `OLED_I2C_HZ` to 100000.
* Upload fails: unplug LEFT/RIGHT buttons' presses (GPIO0/GPIO2) during reset.
* Too fast/slow: tweak constants at the top of each `game_*.cpp`.
