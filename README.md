# 🐉 so_long

A 2D mini-game built with **MiniLibX** as part of the **42 School** curriculum.
Your mission: parse a map, validate it, load textures, and allow the player to move through the world collecting items before exiting.

---

## 📘 Table of Contents

1. [Introduction](#-introduction)
2. [Project Overview](#-project-overview)
3. [How It Works](#%EF%B8%8F-how-it-works)
4. [Map Parsing & Validation](#%EF%B8%8F-map-parsing--validation)
5. [Rendering Pipeline](#-rendering-pipeline)
6. [Player Movement](#-player-movement)
7. [Program Flow](#-program-flow)
8. [Error Handling](#-error-handling)
9. [Compilation](#%EF%B8%8F-compilation)
10. [Usage](#%EF%B8%8F-usage)
11. [Project Structure](#-project-structure)

---

## 🧠 Introduction

**so_long** is a small 2D game where you move a character through a map made of tiles:

* **Walls**
* **Items (collectibles)**
* **A single player**
* **A single exit**

The game focuses on:

* Parsing files and validating game rules
* Using **MiniLibX** to draw sprites
* Handling input events
* Memory management and cleanup
* Basic game loop logic

This repository also includes a **bonus version** featuring enemies and animations.

---

## 📦 Project Overview

The game receives a single `.ber` map file.
The core responsibilities are:

* Parsing the map from disk
* Verifying the map is valid (walls, characters, rectangular shape)
* Checking path solvability using a duplicated copy of the map
* Rendering the game world with MLX
* Handling movement with animations
* Detecting win and loss conditions
* Managing dynamic memory cleanly

The main function reflects this straightforward flow:
It validates input, parses the map, initializes the game, renders it and frees everything afterward.

---

## ⚙️ How It Works

The game revolves around two main structures:

### **`t_map`**

* Raw map array (`map`)
* Copy of the map (`map_cpy`) for flood-fill validation
* Dimensions (`width`, `height`)
* Pointer to player coordinates
* Pointer to exit coordinates

Initialized in `ft_initialize_map()` by loading the map, duplicating it, and extracting positions.

---

### **`t_game`**

Holds everything needed for gameplay:

* MLX instance and window
* Calculated window size:

  ```
  win_width = map->width * 16  
  win_height = map->height * 16
  ```

  Based on tiles of **16x16 pixels**
* Step counter
* Item count
* Exit open state
* Map reference

Initialized inside `ft_initialize_game()`.

---

## 🗺️ Map Parsing & Validation

### Map loading

Maps are read line-by-line using `get_next_line`, allocated dynamically, and stored in `map->map`.

Dimensions are then computed using `ft_get_map_size()` which counts characters and lines.

---

### Validation Rules

**Handled inside `ft_map_valid()`**:

* Map must not be empty
* Map must be **rectangular** (all lines same length)
* Map must be fully **surrounded by walls** (`1`) on all edges
* Exactly **1 player**, **1 exit**, and at least **1 item** must exist
* No duplicates of player or exit allowed

Additionally, the copied map (`map_cpy`) is used to verify that all items and exit are reachable.

---

## 🎨 Rendering Pipeline

Rendering begins in the main render function:
`ft_render_game()` creates the MLX window and registers input hooks:

```c
game->mlx = mlx_init();
game->window = mlx_new_window(...);
mlx_key_hook(game->window, ft_move_player, game);
mlx_hook(game->window, ON_DESTROY, 0, ft_close_game, game);
mlx_loop(game->mlx);
```

---

### Tile-by-tile rendering

`ft_render_map()` draws:

1. **Floor tiles**

   * `"./textures/floor.xpm"`
   * Drawn either for the whole map or for a specific tile after movement

2. **Walls & Exit**

   * `"./textures/wall.xpm"`
   * `"./textures/exit.xpm"` once opened

3. **Items**

   * `"./textures/chest.xpm"`
   * One sprite per item tile

4. **Player**

   * Animated idle/walk sprites depending on direction
   * Loaded using `mlx_xpm_file_to_image` inside movement functions

Everything is drawn using `mlx_put_image_to_window`.

---

## 🏃 Player Movement

Player movement is handled via `mlx_key_hook`.
Movement actions include:

* Up
* Down
* Left
* Right

Each direction has:

* Collision checking
* Sprite animation (idle/walk)
* Item pickup detection
* Exit unlocking
* Step counter increment
* Re-rendering

Example from **move up** logic:
Checks for walls and exit, loads sprite, applies movement, updates steps, prints to terminal.

Exit becomes active once all items are collected.

In the **bonus version**, enemies are also considered, with hit detection and animations.

---

## 🔄 Program Flow

Complete lifecycle:

```
Start
↓
Validate arguments (must be 1 .ber file)
↓
ft_map_parser → Load map into memory
↓
ft_map_valid → Rectangular, walls, exits, items, player
↓
ft_initialize_game → Create game struct, calc window size
↓
ft_render_game → Create window and render map
↓
Game Loop:
   - Wait for key input
   - Move player
   - Redraw affected tiles
   - Check win/lose conditions
↓
Free all memory
↓
Exit
```

Main entry point in `so_long.c` confirms this flow clearly.

---

## 🚨 Error Handling

Common error conditions:

* Invalid number of arguments
* Invalid file extension
* Map cannot be read
* Map violates validation rules (walls, shape, missing player, etc.)
* Image loading errors (bad texture path)
* Memory allocation failure

Errors are usually handled by:

* `ft_exit_program(...)`
* `ft_free_map()` / `ft_free_game()` for cleanup

Program aborts gracefully with all allocated memory freed.

---

## 🛠️ Compilation

The Makefile builds both **mandatory** and **bonus** versions.

### Targets

* `make` → builds `so_long`
* `make bonus` → builds `so_long_bonus`
* `make clean` → removes objects
* `make fclean` → removes binaries
* `make re` → full rebuild

### Dependencies

* **libft** (`lib/libft/`)
* **MiniLibX** (`lib/miniLibX/`)

The Makefile compiles all sources listed under:

* `src/*/*.c` for mandatory
* `bonus/src/*/*.c` for bonus

Sprites and windowing rely on:

```
-lmlx -framework OpenGL -framework AppKit
```

As shown in the Makefile's LFLAGS rule.

---

## ▶️ Usage

Run:

```sh
./so_long map.ber
```

Moves:

* **W / ↑** → up
* **S / ↓** → down
* **A / ←** → left
* **D / →** → right

Win condition:

* Collect all items → exit appears → walk onto the exit

Bonus adds:

* Enemies
* Player/enemy animations
* Additional map logic

---

## 📁 Project Structure

Based on the Makefile and file discovery:
(Non-bonus version shown)

```
.
├── inc/
│   ├── so_long.h
│   ├── map_utils.h
│   ├── ...
├── src/
│   ├── main/
│   │   ├── so_long.c
│   │   ├── ft_render_game.c
│   │   ├── ft_render_map.c
│   │   ├── player_movement.c
│   ├── utils/
│   │   ├── map_utils.c
│   │   ├── game_utils.c
│   │   ├── free_utils.c
│   │   ├── position_utils.c
│   │   ├── steps_utils.c
│   ├── parsing/
│   │   ├── ft_map_parser.c
│   │   ├── ft_map_valid.c
│   │   ├── file_utils.c
│   │   ├── ft_map_valid_utils.c
├── bonus/
│   ├── src/main/
│   ├── src/utils/
│   ├── src/enemy/
├── textures/
├── lib/
│   ├── libft/
│   └── miniLibX/
└── Makefile
```
