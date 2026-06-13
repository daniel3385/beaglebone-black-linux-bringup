# Creating a Cloned Ubuntu Instance in WSL2

This guide explains how to create a separate Ubuntu instance in WSL2 for testing, experimentation, and development without affecting your primary environment.

---

## Clone Your Existing Ubuntu Installation

### 1. Check Installed WSL Distributions

Open PowerShell and run:

```powershell
wsl --list --verbose
```

Example output:

```text
Ubuntu-22.04
```

---

### 2. Export Your Existing Distribution

Create a backup of your current Ubuntu installation as a `.tar` archive:

```powershell
wsl --export Ubuntu-22.04 C:\wsl\ubuntu-base.tar
```

> If the `C:\wsl` directory does not exist, create it first.

---

### 3. Import the Backup as a New Distribution

Create a separate WSL instance that will be used as a development or laboratory environment:

```powershell
wsl --import Ubuntu-Lab C:\wsl\Ubuntu-Lab C:\wsl\ubuntu-base.tar --version 2
```

This creates a new WSL distribution named:

```text
Ubuntu-Lab
```

---

### 4. Verify the New Distribution

Run:

```powershell
wsl --list --verbose
```

You should see something similar to:

```text
Ubuntu-22.04
Ubuntu-Lab
```

---

### 5. Launch the New Environment

Start the cloned instance:

```powershell
wsl -d Ubuntu-Lab
```

You now have an isolated environment where you can freely test software, toolchains, kernels, or system configurations without affecting your main Ubuntu installation.

---

## Restoring the Environment

If the lab environment becomes unusable or you simply want to start over:

```powershell
wsl --unregister Ubuntu-Lab
```

This removes the cloned instance completely.

You can then recreate it by repeating the import step using the original backup archive.

---

## Use Cases

* Testing cross-compilation toolchains
* Kernel development
* Embedded Linux experiments
* Buildroot and Yocto evaluations
* Package installation testing
* Learning and troubleshooting without risking your primary environment

This approach provides a lightweight and reproducible Linux sandbox on Windows using WSL2.
