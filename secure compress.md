# 🔐 `secure_compress` — Secure Archiving with Encryption

## 📖 Overview

`secure_compress` is a shell function for securely archiving and encrypting a folder or file in one step. It uses:

- `tar` + `xz` for high-compression archival
- [`age`](https://age-encryption.org/) for modern, simple encryption with passphrase support

The resulting file is:

```
.tar.xz.age
```

Encrypted. Portable. Clean.

---

## 🧪 Requirements

Make sure the following tools are installed:

- `tar`
- `xz`
- `age` (must support passphrase mode: check with `age --version`)
- (Optional) `read`, `basename`, and standard UNIX tools (your shell should already have these)

---

## 🚀 Usage

```bash
secure_compress <target> [output_name]
````

- `target`: File or directory to archive and encrypt
    
- `output_name` (optional): Custom name for archive (without extension)
    

### Examples

```bash
secure_compress notes
# → produces: notes.tar.xz.age

secure_compress project build_2025_backup
# → produces: build_2025_backup.tar.xz.age
```

---

## 🔐 How it Works

1. Compresses the input into a `.tar.xz` archive:
    
    ```bash
    tar -cf - "$input" | xz -9 > "$output.tar.xz"
    ```
    
2. Prompts for a passphrase and encrypts the archive:
    
    ```bash
    age -p -o "$output.tar.xz.age" "$output.tar.xz"
    ```
    
3. Deletes the intermediate `.tar.xz` file, leaving only:
    
    ```
    output.tar.xz.age
    ```
    

---

## ✅ Features

- 🔒 **Strong encryption** (AES-256 via `age`)
    
- 🗃️ **Compression** (xz level 9)
    
- 🧹 **Cleanup** of plaintext archive after encryption
    
- 🚫 **Overwrite protection** for existing `.age` archives
    
- 💻 Works on macOS, Linux, WSL, Termux
    

---

## 🧠 Tips

- The output `.age` file is completely encrypted — even filenames are hidden inside the archive.
    
- You can store the `.age` file anywhere (cloud, USB, etc.) without exposing contents.
    
- Use with `secure_extract` to safely restore the archive (see below).
    

---

## 🔓 Decrypt & Extract (Companion)

You can decrypt manually:

```bash
age -d -i - archive.tar.xz.age > archive.tar.xz
tar -xJf archive.tar.xz
```

Or define a `secure_extract` function that automates this (ask if you want it written).

---

## ⚠️ Warnings

- The archive is encrypted with **a password you provide interactively**. If you forget it, the archive is irrecoverable.
    
- This script does not currently support keyfile-based encryption (though `age` supports it).
    
- If interrupted, the temporary `.tar.xz` file may remain on disk — delete it manually if needed.
    

---

## 🛠️ Customisation

You can easily tweak:

- Compression level (currently `xz -9`)
    
- Encryption method (e.g., use `gpg` instead of `age`)
    
- Output folder
    
- Add shredding for `.tar.xz` (if paranoid)
    

---

## 📄 License

This function is unlicensed, free to copy, modify, and weaponise as needed.

---

## 🧨 Secure Compress Function (for `.zshrc/.bashrc` or script)

```shell
secure_compress () {
    local input="$1"
    local base dest archive tmp_encrypted

    if [[ -z "$input" || ! -e "$input" ]]; then
        echo "❌ Usage: secure_compress <file-or-folder> [output_name]"
        return 1
    fi

    base="$(basename "$input")"
    dest="${2:-$base}"
    archive="$dest.tar.xz"
    tmp_encrypted="$archive.age"

    if [[ -f "$tmp_encrypted" ]]; then
        echo "⚠️ '$tmp_encrypted' already exists. Overwrite? (y/N)"
        read -r confirm
        [[ "$confirm" != "y" ]] && echo "❌ Aborted." && return 1
        rm -f "$tmp_encrypted"
    fi

    echo "🛠️  Compressing $input → $archive..."
    tar -cf - "$input" | xz -9 > "$archive"

    echo -n "🔑 Enter encryption password: "
    read -rs pass
    echo

    echo "$pass" | age -p -o "$tmp_encrypted" "$archive"
    rm -f "$archive"

    echo "✅ Done: $tmp_encrypted"
}
```