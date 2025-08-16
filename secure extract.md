# 🔓 `secure_extract` — Decrypt and Extract `.tar.xz.age` Archives

## 📖 Overview

`secure_extract` is a shell function designed to reverse the operation of `secure_compress`. It:

- Decrypts a `.tar.xz.age` archive using a passphrase
- Extracts the archive contents into a specified or current directory
- Securely deletes the decrypted `.tar.xz` file after extraction

Use it to recover encrypted backups or folders you previously compressed using `secure_compress`.

---

## 🧪 Requirements

- [`age`](https://age-encryption.org/) — must support passphrase decryption
- `tar` and `xz` — for archive extraction
- Shell with standard utilities (`read`, `basename`, etc.)

---

## 🚀 Usage

```bash
secure_extract <archive.tar.xz.age> [destination]
```

- `archive.tar.xz.age`: the encrypted archive to decrypt and extract
    
- `destination` (optional): the directory to extract to (defaults to `.`)
    

### Examples

```bash
secure_extract secrets.tar.xz.age
# → Extracts to current directory

secure_extract backup_2025.tar.xz.age /Users/osas/Recovered
# → Extracts into /Users/osas/Recovered
```

---

## 🔐 What it Does

1. Prompts you for a decryption passphrase
    
2. Decrypts the `.age` file into a temporary `.tar.xz` archive:
    
    ```bash
    age -d -i - file.age > file.tar.xz
    ```
    
3. Extracts the archive using:
    
    ```bash
    tar -xJf file.tar.xz -C destination
    ```
    
4. Deletes the temporary `.tar.xz` after successful extraction
    

---

## ✅ Features

- 🔐 Password-based decryption (no key files needed)
    
- 🧼 Cleans up decrypted `.tar.xz` after extraction
    
- 📂 Optional destination parameter
    
- 🔄 Pairs seamlessly with `secure_compress`
    

---

## ⚠️ Warnings

- If you enter the wrong password, decryption will fail and the process aborts cleanly.
    
- If extraction fails after decryption, the `.tar.xz` file remains — remove it manually if needed.
    
- This function assumes the `.age` file wraps a `.tar.xz` archive. Do not use on arbitrary `.age` files.
    

---

## 🛠️ Customisation

- To shred the temporary `.tar.xz` file instead of deleting:  
    Replace `rm -f "$tmp"` with:
    
    ```bash
    shred -u "$tmp"
    ```
    
- To extract silently, add `-q` to `tar -xJf`.
    

---

## 🧨 Function Definition (for `.zshrc` or script)

```bash
secure_extract () {
    local archive="$1"
    local tmp="${archive%.age}"
    local dest="${2:-.}"

    if [[ -z "$archive" || ! -f "$archive" ]]; then
        echo "❌ Usage: secure_extract <file.tar.xz.age> [destination]"
        return 1
    fi

    if [[ "${archive##*.}" != "age" ]]; then
        echo "❌ Not an .age file: $archive"
        return 1
    fi

    echo -n "🔑 Enter decryption password: "
    read -rs pass
    echo

    echo "🧨 Decrypting $archive..."
    echo "$pass" | age -d -i - "$archive" > "$tmp" || {
        echo "❌ Decryption failed."
        rm -f "$tmp"
        return 1
    }

    echo "📦 Extracting archive to '$dest'..."
    mkdir -p "$dest"
    tar -xJf "$tmp" -C "$dest" || {
        echo "❌ Extraction failed."
        rm -f "$tmp"
        return 1
    }

    rm -f "$tmp"
    echo "✅ Done: extracted to '$dest'"
}
```

---

## 📄 License

Free to use, extend, or weaponise. Surveillance-proof? No. But a good start.

---

## 🧠 Pair With

- `secure_compress`: for creating encrypted `.tar.xz.age` archives
    

---

## 🗨️ Questions?

The machine doesn't forget.