Source: https://docs.kernel.org/rust/quick-start.html

Attempted on:
Ubuntu 24.04 LTS

# Extra pre-req not mentioned
sudo apt update && sudo apt install -y \
    build-essential \
    libncurses-dev \
    bison \
    flex \
    libssl-dev \
    libelf-dev \
    bc \
    dwarves \
    libdw-dev \
    git \
    curl \
    llvm \
    clang \
    lld \
    gawk

# Install Rustup, the official Rust toolchain manager, with its default Rust toolchain.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

# Load Rustup's Cargo environment variables into the current terminal session.
source $HOME/.cargo/env

# Display the currently active Rust compiler version.
rustc --version

# Ubuntu 24.04: Install the Rust 1.85 toolchain, Rust source, bindgen 0.71, rustfmt, and Clippy.
sudo apt install rustc-1.85 rust-1.85-src bindgen-0.71 rustfmt-1.85 rust-1.85-clippy

# Make rustfmt-1.85 available under the versioned command expected by the kernel build.
sudo ln -s /usr/lib/rust-1.85/bin/rustfmt /usr/bin/rustfmt-1.85

# Make clippy-driver-1.85 available under the versioned command expected by the kernel build.
sudo ln -s /usr/lib/rust-1.85/bin/clippy-driver /usr/bin/clippy-driver-1.8

# Clone Rust-for-Linux Github Project
cd ~/
git clone --depth=1 https://github.com/Rust-for-Linux/linux.git rust-linux
cd ~/rust-linux

# Check that the Rust toolchain satisfies the kernel's Rust requirements.
make LLVM=1 rustavailable

# Configure the kernel build directory to use the stable Rust toolchain when using rustup.
rustup override set stable

# Install the Rust standard library source required to cross-compile core for the kernel.
rustup component add rust-src

# Install bindgen for generating Rust bindings to the kernel's C APIs.
cargo install --locked bindgen-cli

# Display the installed bindgen version.
bindgen --version

# Install rustfmt if it is not already installed by the active rustup profile.
rustup component add rustfmt

# Install Clippy if it is not already installed by the active rustup profile.
rustup component add clippy

# Enable Rust support and Rust sample modules through the kernel configuration interface.
make LLVM=1 menuconfig
General setup -> Rust support (enable)
Kernal hacking -> Sample kernal code (enable) -> Rust samples (enable) -> Minimal + Printing macros (enabled) for now.
Press / -> Search "SYSTEM_TRUSTED_KEYS" -> Press 1 -> Press Enter on "Additional X.509 keys for default system keyring" -> Backspace all text so that it is blank -> exit (You should see something like this: () Additional X.509 keys for default system keyring)
Press / -> Search "SYSTEM_REVOCATION_KEYS" -> Press 1 -> Press Enter -> Backspace all text so that it is blank -> exit

Save and exit

# Verify .config
grep -E '^CONFIG_RUST=|^CONFIG_RUST_IS_AVAILABLE=|^CONFIG_SAMPLES_RUST' .config
grep -E 'CONFIG_SYSTEM_(TRUSTED|REVOCATION)_KEYS' .config

# Generate rust-project.json for rust-analyzer support.
make LLVM=1 rust-analyzer
ls -lh rust-project.json
head -20 rust-project.json <- This command may take time

# Build the Linux kernel using the complete LLVM toolchain.
make LLVM=1
# OR USE MORE CORES
make LLVM=1 -j"$(nproc)"
