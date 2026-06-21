# Developer Environment Setup

macOS Apple Silicon (arm64) — fully reproducible recipe.

**Versions installed:**
Python 3.13.14 · Node.js 24.17.0 LTS · Go 1.26.4 · Rust 1.96.0 · PlatformIO 6.1.19 · ESP-IDF v6.0.1 · Pico SDK 2.2.0 · arm-none-eabi-gcc 16.1.0 · VS Code 1.125.1

---

## Prerequisites

Homebrew must already be installed. If not:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Also ensure Xcode Command Line Tools are present (provides clang, git, make):

```bash
xcode-select --install
```

---

## 1. Create directory structure

```bash
mkdir -p ~/dev ~/sdk
```

- `~/dev/` — all projects live here, one subdirectory per project
- `~/sdk/` — SDK installations (ESP-IDF, Pico SDK)

---

## 2. Install Homebrew packages

```bash
brew install xz pyenv fnm go cmake ninja dfu-util arm-none-eabi-gcc
brew install --cask visual-studio-code
```

**What each provides:**
- `xz` — required by pyenv to compile Python with lzma support (must be installed *before* Python)
- `pyenv` — Python version manager
- `fnm` — fast Node.js version manager
- `go` — Go toolchain
- `cmake`, `ninja` — build system used by ESP-IDF and Pico SDK
- `dfu-util` — firmware flashing over USB DFU
- `arm-none-eabi-gcc` — ARM bare-metal cross-compiler for Pico SDK
- `visual-studio-code` — code editor

---

## 3. Python (pyenv + uv)

Install Python 3.13 and set it as the global default:

```bash
pyenv install 3.13
pyenv global 3.13
```

Verify lzma is working (must not raise an ImportError):

```bash
python3 -c "import lzma; print('ok')"
```

Install uv (fast Python package/project manager):

```bash
pip install uv
```

---

## 4. Node.js (fnm)

```bash
eval "$(fnm env)"
fnm install --lts
fnm default lts-latest
```

---

## 5. Rust (rustup)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path
```

The `--no-modify-path` flag keeps rustup from editing shell files — the `~/.zshrc` we write later handles this explicitly.

---

## 6. PlatformIO

Install into the pyenv-managed Python (make sure pyenv shims are active):

```bash
pip install platformio
```

---

## 7. ESP-IDF v6.0.1

Clone with all submodules, check out the stable release, then run the installer:

```bash
git clone --recursive https://github.com/espressif/esp-idf.git ~/sdk/esp-idf
cd ~/sdk/esp-idf
git checkout v6.0.1
git submodule update --init --recursive
./install.sh all
```

`install.sh all` downloads Espressif's cross-compilers (Xtensa and RISC-V) and sets up an isolated Python venv under `~/.espressif/`. It takes several minutes and ~500 MB.

---

## 8. Raspberry Pi Pico SDK

```bash
git clone https://github.com/raspberrypi/pico-sdk.git ~/sdk/pico-sdk
cd ~/sdk/pico-sdk
git submodule update --init
```

---

## 9. Shell configuration (~/.zshrc)

Create `~/.zshrc` with the following content:

```zsh
# ── Homebrew ──────────────────────────────────────────────────────────────────
eval "$(/opt/homebrew/bin/brew shellenv)"

# ── pyenv (Python) ────────────────────────────────────────────────────────────
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# ── fnm (Node.js) ─────────────────────────────────────────────────────────────
eval "$(fnm env --use-on-cd --shell zsh)"

# ── Go ────────────────────────────────────────────────────────────────────────
export GOPATH="$HOME/go"
export PATH="$PATH:$GOPATH/bin"

# ── Rust ──────────────────────────────────────────────────────────────────────
source "$HOME/.cargo/env"

# ── ESP-IDF ───────────────────────────────────────────────────────────────────
# Run `get_idf` in a project shell to activate the ESP-IDF environment
alias get_idf='. "$HOME/sdk/esp-idf/export.sh"'

# ── Raspberry Pi Pico SDK ─────────────────────────────────────────────────────
export PICO_SDK_PATH="$HOME/sdk/pico-sdk"

# ── PlatformIO ────────────────────────────────────────────────────────────────
export PATH="$PATH:$HOME/.platformio/penv/bin"
```

Open a new terminal for these settings to take effect.

---

## 10. VS Code extensions

Install sequentially to avoid race conditions:

```bash
code --install-extension ms-python.python --force
code --install-extension golang.Go --force
code --install-extension rust-lang.rust-analyzer --force
code --install-extension platformio.platformio-ide --force
code --install-extension espressif.esp-idf-extension --force
code --install-extension raspberry-pi.raspberry-pi-pico --force
```

**Extensions installed:**
- `ms-python.python` — Python (pulls in Pylance and Debugpy automatically)
- `golang.Go` — Go language support
- `rust-lang.rust-analyzer` — Rust language server
- `platformio.platformio-ide` — PlatformIO (pulls in ms-vscode.cpptools)
- `espressif.esp-idf-extension` — ESP-IDF project management and flashing
- `raspberry-pi.raspberry-pi-pico` — Pico SDK project wizard and flashing

---

## How to use the embedded toolkits

### ESP-IDF
The ESP-IDF environment is not active by default (it overrides PATH in ways that conflict with other tools). Activate it per-session when working on ESP projects:

```bash
get_idf
```

Then use `idf.py` as normal: `idf.py build`, `idf.py flash`, etc.

### Pico SDK
`PICO_SDK_PATH` is always set. Any CMake project that includes `pico_sdk_import.cmake` picks it up automatically. Build as usual:

```bash
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### PlatformIO
Use the `pio` CLI or the PlatformIO sidebar in VS Code. Projects are isolated via `platformio.ini`.

### Python project isolation
Each project gets its own virtualenv managed by `uv`:

```bash
cd ~/dev/my-project
uv init          # creates pyproject.toml
uv venv          # creates .venv
uv add requests  # adds a dependency
```

### Node.js project isolation
Pin a Node version per project with `.node-version`:

```bash
echo "24" > .node-version   # fnm activates it automatically on cd
```

### Go and Rust
Both have built-in module/package isolation — `go.mod` and `Cargo.toml` per project. No extra setup needed.
