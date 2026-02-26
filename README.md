name: Build ICE-Guard (Windows EXE + Linux DEB)

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

jobs:

  # -------------------------
  # WINDOWS BUILD (EXE)
  # -------------------------
  build-windows:
    name: Build Windows EXE
    runs-on: windows-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pyinstaller

      - name: Build ICE-Guard.exe
        run: |
          pyinstaller --noconsole --onefile blackice/main.py --name ICE-Guard

      - name: Upload Windows EXE artifact
        uses: actions/upload-artifact@v4
        with:
          name: ICE-Guard-Windows
          path: dist/ICE-Guard.exe


  # -------------------------
  # LINUX BUILD (DEB)
  # -------------------------
  build-linux:
    name: Build Linux Binary + DEB
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          sudo apt update
          sudo apt install -y dpkg-dev
          pip install -r requirements.txt
          pip install pyinstaller

      - name: Build ice-guard binary
        run: |
          pyinstaller --onefile blackice/main.py --name ice-guard

      - name: Prepare DEB structure
        run: |
          mkdir -p iceguard_deb/DEBIAN
          mkdir -p iceguard_deb/usr/local/bin
          cp dist/ice-guard iceguard_deb/usr/local/bin/
          chmod 755 iceguard_deb/usr/local/bin/ice-guard

          cat <<EOF > iceguard_deb/DEBIAN/control
          Package: ice-guard
          Version: 1.0
          Section: utils
          Priority: optional
          Architecture: amd64
          Maintainer: Christoffer
          Description: ICE-Guard network intrusion monitor
           ICE-Guard is a simplified, educational intrusion detection tool inspired by BlackICE.
          EOF

      - name: Build DEB package
        run: |
          dpkg-deb --build iceguard_deb
          mv iceguard_deb.deb ice-guard_1.0_amd64.deb

      - name: Upload Linux DEB artifact
        uses: actions/upload-artifact@v4
        with:
          name: ICE-Guard-DEB
          path: ice-guard_1.0_amd64.deb
