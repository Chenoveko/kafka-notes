# ⚡ Apache Kafka Notes

📖 Read the book online at: [Github Pages](https://chenoveko.github.io/kafka-notes/)

## 🛠️ Build the Book Locally

### 1. Install Rust with `rustup`

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### 2. Verify the Rust Installation
```bash
rustc --version
cargo --version
```

### 4. Install the required system packages
```bash
sudo apt update
sudo apt install build-essential -y
```

### 5. Install mdBook wit Cargo
```bash
cargo install mdbook
```

### 6. Verify the mdBook Installation
```bash
mdbook --version
```

### 7. Clone the repository
```bash
git clone https://github.com/Chenoveko/kafka-notes.git
```

### 8. Build and serve the book
Run this command inside the cloned repository:
```bash
mdbook serve --open -p 8080
```
The book will be available locally, usually at:
```bash
http://localhost:8080
```

### 9. Clean Enviroment
To remove the generated book directory, run:
```bash
mdbook clean
```
