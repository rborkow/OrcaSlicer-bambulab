<div align="center">

<picture>
  <img alt="OrcaSlicer logo" src="resources/images/OrcaSlicer.png" width="15%" height="15%">
</picture>

## This version of OrcaSlicer restores full BambuNetwork support for Bambu Lab printers.

You are not limited to LAN only.  
It works over the internet just like before, through BambuNetwork, with full functionality for normal use and printing.

## Installation

### Windows

Windows requires WSL 2.

Before first launch, open Command Prompt or PowerShell as Administrator and run:

```bat
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

Restart Windows, then launch Orca Studio.

### Linux

On Linux, a normal installation is enough.

### macOS

Requires macOS 11.3 or newer (Apple Silicon or Intel) and roughly 4 GB of
free disk space for the bundled Linux VM that hosts the Bambu network plugin.

1. Download `OrcaSlicer_Mac_universal_*.dmg` from the
   [Releases](../../releases) page.
2. Mount the DMG and drag `OrcaSlicer.app` into `/Applications`.
3. The DMG is currently unsigned, so on first launch macOS will block it.
   Open **System Settings → Privacy & Security**, scroll to the bottom, and
   click **Open Anyway**.
4. Open Terminal and install the network bridge runtime (this fetches Lima
   via Homebrew if installed, otherwise downloads it into Application
   Support, and provisions a Linux VM named `orcaslicer-bambu-network`):

   ```bash
   /Applications/OrcaSlicer.app/Contents/MacOS/install_runtime_macos.sh \
       -PluginDir /Applications/OrcaSlicer.app/Contents/MacOS
   ```

   On Apple Silicon you may also be prompted to install Rosetta — accept.
5. Optional sanity check:

   ```bash
   /Applications/OrcaSlicer.app/Contents/MacOS/verify_runtime_macos.sh
   ```
6. Launch OrcaSlicer.

The Lima VM auto-starts at login. Runtime files and logs live under
`~/Library/Application Support/OrcaSlicer/macos-bridge/`.


## BMCU

I also encourage you to use BMCU.

You can find BMCU firmware in my repositories.

</div>
