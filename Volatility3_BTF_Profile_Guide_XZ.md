# Guide: Creating a Linux Kernel Symbol Profile for Volatility 3 Using BTF

This guide explains how to create a Linux kernel symbol profile for **Volatility 3** using **BTF (BPF Type Format)** information.

---

## Prerequisites

- Linux system with access to the target kernel files.
- Python environment suitable for running Volatility 3.
- Internet access to download required tools.
- Access to the following artifacts corresponding to the system from which the memory image was acquired:
  - `vmlinuz-*` for the kernel that was running when the memory image was captured.
  - `System.map-*` for the same kernel version.
  - The complete kernel banner extracted from the memory image being analyzed.

> **Important:** The `vmlinuz`, `System.map`, and kernel banner must all originate from the same kernel build that was running when the memory image was acquired.

---

## Step 1 – Install Volatility 3

```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
```

---

## Step 2 – Patch Volatility 3 to Support BTF Symbols

Edit:

```text
volatility3/volatility3/schema/schema-6.2.0.json
```

Replace:

```json
"pattern": "^(dwarf|symtab|system-map)$"
```

With:

```json
"pattern": "^(btf|symdb|dwarf|symtab|system-map)$"
```

---

## Step 3 – Collect Required Kernel Files

From `/boot` copy:

```text
vmlinuz-<kernel-version>
System.map-<kernel-version>
```

Example:

```text
vmlinuz-6.1.128-1.el9.x86_64
System.map-6.1.128-1.el9.x86_64
```

> **Important:** Ensure that the files match the kernel that was running when the memory image was acquired.

---

## Step 4 – Extract the Uncompressed Kernel (vmlinux)

```bash
wget -O extract-vmlinux https://raw.githubusercontent.com/torvalds/linux/master/scripts/extract-vmlinux
chmod +x extract-vmlinux
./extract-vmlinux vmlinuz-6.1.128-1.el9.x86_64 > vmlinux-6.1.128-1.el9.x86_64
```

---

## Step 5 – Download btf2json

```bash
wget https://github.com/vobst/btf2json/releases/download/v0.1.0/btf2json
chmod +x btf2json
```

---

## Step 6 – Obtain the Kernel Banner

```bash
python3 vol.py -f memory-image.raw linux.banners.Banners
```

Example banner:

```text
Linux version 6.1.128-1.el9.x86_64
(builduser@example.org)
(gcc (GCC) 11.5.0, GNU ld version 2.35.2)
#1 SMP PREEMPT_DYNAMIC Tue Jan 14 12:00:00 UTC 2026
```

---

## Step 7 – Generate the Volatility Symbol File

```bash
./btf2json \
--btf /path/to/vmlinux-6.1.128-1.el9.x86_64 \
--map /path/to/System.map-6.1.128-1.el9.x86_64 \
--banner "Linux version 6.1.128-1.el9.x86_64 (builduser@example.org) (gcc (GCC) 11.5.0, GNU ld version 2.35.2) #1 SMP PREEMPT_DYNAMIC Tue Jan 14 12:00:00 UTC 2026" \
| xz -9 > Linux-6.1.128-1.el9.x86_64.json.xz
```

---

## Step 8 – Install the Symbol File

```bash
cp Linux-6.1.128-1.el9.x86_64.json.xz volatility3/volatility3/symbols/
```

---

## Step 9 – Verify the Profile

```bash
python3 vol.py isfinfo
```
