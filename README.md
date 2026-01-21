# ovpntmp DEPRICATED
# This has been moved to https://github.com/GustavPetterssonBjorklund/scripts

# Installation
```bash
mkdir -p ~/.local/bin
cat << 'EOF' > ~/.local/bin/ovpntmp
#!/usr/bin/env bash

set -euo pipefail

OVPN_DIR="$HOME/Downloads"

# Icons
OK="✔ OK"
RUN="▶ RUNNING"
FAIL="✖ FAILED"
INFO="ℹ"

# Collect .ovpn files
mapfile -t OVPN_FILES < <(find "$OVPN_DIR" -maxdepth 1 -type f -name '*.ovpn' | sort)

if (( ${#OVPN_FILES[@]} == 0 )); then
  echo "$FAIL No .ovpn files found in $OVPN_DIR"
  exit 1
fi

chosen_file=""

if (( ${#OVPN_FILES[@]} == 1 )); then
  chosen_file="${OVPN_FILES[0]}"
  echo "$INFO Found single .ovpn:"
  echo "    $chosen_file"
else
  echo "$INFO Multiple .ovpn files found:"
  for i in "${!OVPN_FILES[@]}"; do
    printf "  %2d) %s\n" "$((i+1))" "${OVPN_FILES[$i]}"
  done
  echo

  while :; do
    read -rp "Select file (1-${#OVPN_FILES[@]}): " choice
    if [[ "$choice" =~ ^[0-9]+$ ]] && (( choice >= 1 && choice <= ${#OVPN_FILES[@]} )); then
      idx=$((choice-1))
      chosen_file="${OVPN_FILES[$idx]}"
      break
    else
      echo "$INFO Invalid choice, try again."
    fi
  done

  echo
  echo "$OK Selected:"
  echo "    $chosen_file"
  echo

  read -rp "Delete all other .ovpn files? [y/N]: " ans
  ans=${ans:-n}

  if [[ "$ans" =~ ^[Yy]$ ]]; then
    echo "$INFO Cleaning up..."
    for i in "${!OVPN_FILES[@]}"; do
      if (( i != idx )); then
        rm -- "${OVPN_FILES[$i]}" && echo "  $OK removed $(basename "${OVPN_FILES[$i]}")"
      fi
    done
  else
    echo "$INFO Keeping other .ovpn files."
  fi
fi

echo
echo "$RUN Starting OpenVPN with:"
echo "    $chosen_file"
echo

# Run openvpn and detect exit status
if sudo openvpn --config "$chosen_file"; then
  echo
  echo "$OK OpenVPN exited cleanly."
else
  code=$?
  echo
  echo "$FAIL OpenVPN exited with code $code."
  exit $code
fi
EOF

chmod +x ~/.local/bin/ovpntmp

```
