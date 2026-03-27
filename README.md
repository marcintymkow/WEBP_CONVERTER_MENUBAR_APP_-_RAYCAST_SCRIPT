# WebP-Converter-Menubar-App---Raycast-Script# WebP Auto-Converter

Automatic image-to-WebP conversion on macOS. Watches your Screenshots folder and converts new images in the background.

## Features

- **Automatic monitoring** — watches `~/Pictures/Screenshots`
- **WebP conversion** — quality 75% with Squoosh-style `cwebp` settings
- **Originals removed** — after a successful conversion
- **Green Finder tag** — applied to converted `.webp` files
- **macOS notifications** — one per conversion
- **Login startup** — optional LaunchAgent keeps it running in the background

## Requirements

```bash
brew install webp fswatch
```

## Installation

```bash
# 1. Create install directory
mkdir -p ~/.local/share/webp-converter

# 2. Copy the script
cp webp_auto_converter.sh ~/.local/share/webp-converter/
chmod +x ~/.local/share/webp-converter/webp_auto_converter.sh

# 3. Create LaunchAgent (paths expand from $HOME)
mkdir -p ~/Library/LaunchAgents

cat > ~/Library/LaunchAgents/com.marcin.webp-auto-converter.plist << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.marcin.webp-auto-converter</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>$HOME/.local/share/webp-converter/webp_auto_converter.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>$HOME/.local/share/webp-converter/stdout.log</string>
    <key>StandardErrorPath</key>
    <string>$HOME/.local/share/webp-converter/stderr.log</string>
</dict>
</plist>
EOF

# 4. Load the service
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.marcin.webp-auto-converter.plist
```

## Commands

| Action | Command |
|--------|---------|
| Run manually | `~/.local/share/webp-converter/webp_auto_converter.sh` |
| Run in background (no terminal) | `nohup ~/.local/share/webp-converter/webp_auto_converter.sh > /dev/null 2>&1 &` |
| Enable auto-start | `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.marcin.webp-auto-converter.plist` |
| Disable auto-start | `launchctl bootout gui/$(id -u)/com.marcin.webp-auto-converter` |
| Restart service | `launchctl kickstart -k gui/$(id -u)/com.marcin.webp-auto-converter` |
| Status | `launchctl print gui/$(id -u)/com.marcin.webp-auto-converter` |
| Live logs | `tail -f ~/.local/share/webp-converter/stdout.log` |
| Stop process | `pkill -f webp_auto_converter` |

## File locations

| Item | Path |
|------|------|
| Script | `~/.local/share/webp-converter/webp_auto_converter.sh` |
| LaunchAgent | `~/Library/LaunchAgents/com.marcin.webp-auto-converter.plist` |
| stdout log | `~/.local/share/webp-converter/stdout.log` |
| stderr log | `~/.local/share/webp-converter/stderr.log` |
| Conversion log | `~/.local/share/webp-converter/converter.log` |

## Configuration

Edit variables at the top of `webp_auto_converter.sh`:

```bash
WATCH_FOLDER="$HOME/Pictures/Screenshots"  # Folder to watch
WEBP_QUALITY=75                            # WebP quality (0–100)
```

## Supported formats

- JPG / JPEG
- PNG
- GIF
- TIFF / TIF
- BMP

## Conversion settings

`cwebp` flags (Squoosh-style):

| Flag | Value | Purpose |
|------|-------|---------|
| `-q` | 75 | Quality |
| `-m` | 6 | Slower, better compression |
| `-af` | — | Auto filter |
| `-sharp_yuv` | — | Sharper color conversion |

## Uninstall

```bash
# 1. Stop the service
launchctl bootout gui/$(id -u)/com.marcin.webp-auto-converter

# 2. Remove files
rm -rf ~/.local/share/webp-converter
rm ~/Library/LaunchAgents/com.marcin.webp-auto-converter.plist
```

## Troubleshooting

**Does not start at login:**

```bash
launchctl print gui/$(id -u)/com.marcin.webp-auto-converter
cat ~/.local/share/webp-converter/stderr.log
```

(Exit code 126 often means permission or non-executable script.)

**Missing tools:**

```bash
which cwebp && which fswatch
brew install webp fswatch
```

**Restart the service:**

```bash
launchctl bootout gui/$(id -u)/com.marcin.webp-auto-converter 2>/dev/null
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.marcin.webp-auto-converter.plist
```

**Avoid `Documents` for the script:**  
macOS privacy can block automation there. `~/.local/share/` is a safer install location.

## Author

Marcin Tymków

## License

MIT
