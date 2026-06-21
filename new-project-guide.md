# Starting a New Project

This guide walks through creating a new project, publishing it to GitHub, and starting work — for each of the environments available on this machine.

**Installed runtimes:** Python · Node.js · Go · Rust · PlatformIO · ESP-IDF · Pico SDK

---

## Before you begin: GitHub CLI setup (one time only)

Install and authenticate `gh` if not done yet:

```bash
brew install gh
gh auth login
```

Choose **GitHub.com → HTTPS → Login with a web browser** and follow the prompts.

---

## Common workflow (all project types)

Every project follows the same three-step outer wrapper regardless of language:

```
1. Create the GitHub repository
2. Create the project directory and init the language toolchain
3. Connect the two, push, open in VS Code
```

Steps 1 and 3 are identical for all languages — only step 2 changes. The language-specific sections below cover step 2 in detail.

---

## 1. Create the GitHub repository

```bash
gh repo create my-project-name --private --description "Short description"
```

Flags:
- `--private` / `--public` — visibility
- `--description` — optional but useful

This creates the remote repo. Do **not** tick "Initialize repository" or add a README at this point — the local init does that.

---

## 2. Create the project (language-specific)

Pick the section that matches your project type.

---

### Python

```bash
cd ~/dev
mkdir my-project && cd my-project

uv init .               # creates pyproject.toml + hello.py stub
uv venv                 # creates .venv/ (git-ignored by default)
source .venv/bin/activate
```

Add dependencies as needed:

```bash
uv add requests httpx   # runtime deps
uv add --dev pytest ruff  # dev-only deps
```

Start writing code in `src/` or alongside `pyproject.toml`. Run:

```bash
python main.py
pytest
```

`.venv/` is already in the `.gitignore` that `uv init` generates.

---

### Node.js

```bash
cd ~/dev
mkdir my-project && cd my-project

echo "24" > .node-version   # fnm auto-switches Node version on cd
node --version              # verify

npm init -y                 # creates package.json
```

Or for a TypeScript project:

```bash
npm init -y
npm install --save-dev typescript @types/node ts-node
npx tsc --init
```

Add a `.gitignore` entry for `node_modules`:

```bash
echo "node_modules/" >> .gitignore
```

---

### Go

```bash
cd ~/dev
mkdir my-project && cd my-project

go mod init github.com/YOUR_USERNAME/my-project
```

Create `main.go`:

```go
package main

import "fmt"

func main() {
    fmt.Println("hello")
}
```

Run:

```bash
go run .
go build -o my-project .
```

Go modules handle isolation automatically via `go.mod`.

---

### Rust

```bash
cd ~/dev
cargo new my-project
cd my-project
```

`cargo new` creates the full project structure (`src/main.rs`, `Cargo.toml`) and initialises a git repo inside the directory. Skip `git init` below — cargo already did it.

Run:

```bash
cargo run
cargo test
cargo build --release
```

---

### PlatformIO

Create via VS Code (easiest):

1. Open VS Code, click the PlatformIO icon in the sidebar
2. **PlatformIO Home → New Project**
3. Set name, board (e.g. `Espressif ESP32-S3-DevKitC-1`), framework (`Arduino` or `ESP-IDF`)
4. Location: `~/dev/my-project`
5. Click **Finish** and wait for package download

Or via CLI:

```bash
cd ~/dev
mkdir my-project && cd my-project
pio project init --board esp32-s3-devkitc-1
```

Build and upload:

```bash
pio run               # build
pio run --target upload
pio device monitor    # serial output
```

---

### ESP-IDF

Activate the ESP-IDF environment first (required in every new terminal session):

```bash
get_idf
```

Create a project from the hello_world template:

```bash
cd ~/dev
cp -r $IDF_PATH/examples/get-started/hello_world my-project
cd my-project
```

Or start from scratch using the VS Code ESP-IDF extension:

1. Open VS Code → Command Palette (`Cmd+Shift+P`)
2. **ESP-IDF: New Project** → fill in name, target chip, location (`~/dev/`)

Build, flash, monitor:

```bash
idf.py set-target esp32s3     # or esp32, esp32c3, etc.
idf.py build
idf.py -p /dev/cu.usbserial-* flash monitor
```

The port (`/dev/cu.usbserial-*`) auto-expands to the connected device. Use `idf.py -p $(ls /dev/cu.usbserial-* | head -1) flash monitor` if multiple ports exist.

---

### Raspberry Pi Pico W (Pico SDK)

Use the VS Code Raspberry Pi Pico extension (recommended):

1. Open VS Code → Command Palette (`Cmd+Shift+P`)
2. **Raspberry Pi Pico: New Project from Example** — or **New C/C++ Project**
3. Set name (`my-project`), location (`~/dev/`), board (`Pico W`)
4. The extension generates `CMakeLists.txt`, `main.c`, and the toolchain config

Or manually:

```bash
cd ~/dev
mkdir my-project && cd my-project
```

Create `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.13)
include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)
project(my-project C CXX ASM)
pico_sdk_init()
add_executable(my-project main.c)
target_link_libraries(my-project pico_stdlib)
pico_add_extra_outputs(my-project)
```

Build:

```bash
mkdir build && cd build
cmake .. -DPICO_BOARD=pico_w
make -j$(sysctl -n hw.logicalcpu)
```

Flash: hold **BOOTSEL**, plug in USB, then copy the `.uf2` file:

```bash
cp my-project.uf2 /Volumes/RPI-RP2/
```

---

## 3. Connect to GitHub and push

This step is the same for all project types (except Rust — `cargo new` already ran `git init`).

```bash
# Inside ~/dev/my-project

git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/my-project.git
git push -u origin main
```

For Rust (cargo already initialised git, skip `git init`):

```bash
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/my-project.git
git push -u origin main
```

Or use `gh` to add the remote without copy-pasting the URL:

```bash
gh repo set-default YOUR_USERNAME/my-project
git push -u origin main
```

---

## 4. Open in VS Code

```bash
code .
```

VS Code detects the language and activates the relevant extension automatically:
- Python: picks up `.venv` and pyproject.toml
- Go: detects go.mod
- Rust: detects Cargo.toml and starts rust-analyzer
- PlatformIO: shows the PlatformIO toolbar
- ESP-IDF: shows the IDF status bar (chip target, port)
- Pico: shows the Pico SDK status bar

---

## Daily workflow reminder

```bash
cd ~/dev/my-project     # fnm switches Node version automatically

# Python — activate venv if not using uv run
source .venv/bin/activate

# ESP-IDF — activate in every new terminal
get_idf
```

---

## Useful shortcuts

| Task | Command |
|------|---------|
| Create GitHub repo | `gh repo create NAME --private` |
| Open repo in browser | `gh repo view --web` |
| View repo status | `gh repo view` |
| Create a PR | `gh pr create` |
| List open PRs | `gh pr list` |
| Clone an existing repo | `gh repo clone USER/REPO ~/dev/REPO` |
