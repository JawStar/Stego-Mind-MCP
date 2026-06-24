# StegoMind MCP

A steganography and forensics MCP (Model Context Protocol) server for Claude Desktop. Designed to help solve CTF steganography challenges from 50 to 500+ points.

---

## Tools (20 total)

### Original Tools
| Tool | Description |
|---|---|
| `steghide_info` | Check if a file has embedded hidden data |
| `steghide_extract` | Extract hidden data with a known password |
| `steghide_embed` | Hide a file inside another file |
| `steghide_bruteforce` | Bruteforce steghide password (auto wordlist if none given) |
| `hash_lookup` | Get MD5, SHA1, SHA256, SHA512 hashes |
| `file_entropy` | Measure file randomness (detect encryption/compression) |
| `binwalk_scan` | Scan for embedded files and firmware signatures |
| `exiftool_scan` | Read all EXIF and metadata from a file |
| `strings_scan` | Extract readable strings from any binary |
| `yara_scan` | Scan file against YARA rules |

### New Tools
| Tool | Description |
|---|---|
| `zsteg_scan` | PNG/BMP LSB steganography — scans all bit planes and channels |
| `outguess_extract` | JPEG DCT-domain steganography extraction |
| `stegseek_bruteforce` | Ultra-fast steghide cracker (1000x faster, uses rockyou.txt) |
| `audio_spectrogram` | Generate spectrogram image to detect hidden messages in audio |
| `wav_lsb_extract` | Extract LSB hidden data from WAV audio files |
| `png_chunk_scan` | Parse PNG chunks to detect non-standard/hidden chunks |
| `text_stego_detect` | Detect zero-width chars, SNOW whitespace encoding, homoglyphs |
| `pcap_stego_scan` | Detect DNS tunneling, ICMP payloads, HTTP steganography in PCAPs |
| `image_channel_split` | Split image into R/G/B/Alpha channels + bit planes for visual analysis |
| `auto_analyze` | **Master solver** — auto-detects file type and runs full tool chain |

---

## Auto-Analyze Mode

The most powerful feature. Use it when you are stuck on a CTF challenge:

```
Auto analyze /home/user/challenge.jpg
```

Claude will automatically:
1. Detect file type (image / audio / text / PCAP / binary)
2. Run the full appropriate tool chain
3. Return ranked notable findings
4. Surface any flag-like patterns found

### What auto_analyze runs per file type

**Image (PNG/JPG/BMP):**
exiftool → entropy → binwalk → strings → YARA → png_chunk_scan (PNG) →
zsteg (PNG) → channel_split → steghide_info → stegseek → steghide_bruteforce → outguess (JPEG) → hashes

**Audio (WAV/MP3/OGG):**
exiftool → entropy → binwalk → strings → spectrogram → wav_lsb (WAV) → steghide (WAV) → YARA → hashes

**Text/HTML:**
text_stego_detect → strings → entropy → YARA → direct flag search

**PCAP:**
pcap_stego_scan → strings → entropy

**Binary:**
entropy → binwalk → strings → YARA → hashes

---

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/StegoMind-MCP.git
cd StegoMind-MCP
```

### 2. Create virtual environment and install Python dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 3. Install system tools

```bash
# Core tools
sudo apt install steghide exiftool binwalk yara binutils imagemagick tshark sox ffmpeg

# exiftool alternative package name on some distros
sudo apt install libimage-exiftool-perl

# zsteg (requires Ruby)
sudo apt install ruby
gem install zsteg

# outguess
sudo apt install outguess

# stegseek (fast steghide cracker)
# Download from: https://github.com/RickdeJager/stegseek/releases
# Example for Debian/Ubuntu:
wget https://github.com/RickdeJager/stegseek/releases/download/v0.6/stegseek_0.6-1.deb
sudo apt install ./stegseek_0.6-1.deb

# rockyou wordlist (recommended for stegseek)
sudo apt install wordlists
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

### 4. Configure Claude Desktop

Edit `~/.config/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "StegoMind": {
      "command": "/absolute/path/to/StegoMind-MCP/.venv/bin/python",
      "args": [
        "/absolute/path/to/StegoMind-MCP/server.py"
      ]
    }
  }
}
```

Replace `/absolute/path/to/StegoMind-MCP` with the actual path. Find it with:

```bash
realpath ~/StegoMind-MCP
```

### 5. Restart Claude Desktop

```bash
pkill -f "claude" && claude
```

---

## Usage Examples

### Auto mode (recommended for CTF)
```
Auto analyze /home/user/challenge.jpg
```

### Individual tools
```
Use steghide_info on /home/user/image.jpg
Run zsteg scan on /home/user/challenge.png
Extract strings from /home/user/file.bin
Run binwalk scan on /home/user/firmware.bin
Generate spectrogram for /home/user/audio.wav
Bruteforce steghide on /home/user/image.jpg
Scan /home/user/suspicious.exe with YARA rules
Split image channels for /home/user/challenge.png
Detect text steganography in /home/user/file.txt
Analyze PCAP /home/user/capture.pcapng
```

### Notes
- All tools require **absolute file paths** on your local disk
- Do NOT paste/attach files into Claude chat — provide the file path instead
- `auto_analyze` is the recommended starting point for any CTF challenge

---

## YARA Rules

Five rule files included in `rules/`:

| File | Coverage |
|---|---|
| `steganography.yar` | Steghide, OpenStego, SilentEye, Jsteg, LSB anomalies, hidden archives |
| `malware.yar` | Reverse shells, Base64 payloads, Mimikatz, ransomware, webshells, packers |
| `anomaly.yar` | Polyglot files, PE/ELF mismatches, appended data, double extensions |
| `ctf_forensics.yar` | CTF flag formats, PNG hidden chunks, EXIF anomalies, morse/binary strings |
| `sensitive_data.yar` | AWS keys, private keys, hardcoded credentials, Tor addresses |

Add your own `.yar` files to the `rules/` folder and they will be picked up automatically.

---

## Wordlists

Three wordlists included in `wordlists/`:

| File | Focus |
|---|---|
| `ctf_passwords.txt` | CTF-specific terms, leet-speak, nature words |
| `steghide_common.txt` | Most likely steghide defaults |
| `ctf_platforms_and_terms.txt` | Platform names, file types, tool names |

`steghide_bruteforce` auto-merges all `.txt` files in this folder plus generates filename-based guesses. Add any `.txt` wordlist here and it will be used automatically.

---

## Project Structure

```
StegoMind-MCP/
├── server.py                        # MCP server entry point
├── mcp_instance.py                  # Shared FastMCP instance (fixes circular import)
├── requirements.txt
├── claude_desktop_config_example.json
├── core/
│   ├── exceptions.py
│   ├── executor.py
│   ├── logger.py
│   └── validators.py
├── tools/
│   ├── __init__.py
│   ├── auto_analyze.py              # Master CTF solver
│   ├── steghide_bruteforce.py       # Auto wordlist bruteforce
│   ├── stegseek_bruteforce.py       # Ultra-fast cracker
│   ├── zsteg_scan.py                # PNG/BMP LSB
│   ├── outguess_extract.py          # JPEG DCT stego
│   ├── audio_spectrogram.py         # Spectrogram analysis
│   ├── wav_lsb_extract.py           # WAV LSB extraction
│   ├── png_chunk_scan.py            # PNG chunk parser
│   ├── text_stego_detect.py         # Zero-width / SNOW
│   ├── pcap_stego_scan.py           # Network stego
│   ├── image_channel_split.py       # Channel + bit plane split
│   ├── binwalk_scan.py
│   ├── exiftool_scan.py
│   ├── file_entropy.py
│   ├── hash_lookup.py
│   ├── steghide_embed.py
│   ├── steghide_extract.py
│   ├── steghide_info.py
│   ├── strings_scan.py
│   └── yara_scan.py
├── rules/
│   ├── steganography.yar
│   ├── malware.yar
│   ├── anomaly.yar
│   ├── ctf_forensics.yar
│   └── sensitive_data.yar
├── wordlists/
│   ├── ctf_passwords.txt
│   ├── steghide_common.txt
│   └── ctf_platforms_and_terms.txt
├── logs/
│   └── stegomind.log
└── tests/
    ├── test_server_import.py
    └── test_validators.py
```

---

## License

See LICENSE file.
