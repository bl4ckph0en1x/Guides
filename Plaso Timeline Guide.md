# Creating a Forensic Timeline with Plaso on Ubuntu

This guide explains how to install **Plaso** on Ubuntu and generate a forensic timeline from a disk image using **log2timeline.py** and **psort.py**.

---

# What is Plaso?

Plaso (Plaso Langar Að Safna Öllu) is an open-source digital forensics framework used to extract timestamped events from forensic artifacts and create a unified timeline. It is widely used in DFIR (Digital Forensics and Incident Response) investigations to reconstruct user activity, system behavior, and attack timelines.

---

# Installing Plaso on Ubuntu

## Supported Ubuntu Version

Plaso currently supports:

- **Ubuntu 26.04 LTS (Noble)**

> Note: Other Ubuntu versions are not supported at this time.

## Enable the Universe Repository

```bash
sudo add-apt-repository universe
```

## Add the GIFT Stable PPA

```bash
sudo add-apt-repository ppa:gift/stable
```

## Update Package Lists

```bash
sudo apt update
```

## Install Plaso

```bash
sudo apt install plaso-tools
```

## Verify the Installation

Check that the installation completed successfully:

```bash
log2timeline.py --version
```

```bash
psort.py --version
```

Example output:

```text
20250825
```

---

# Preparing the Evidence

Before creating a timeline, ensure you have a forensic disk image available.

Example:

```text
image.dd
```

## Supported Image Formats

Plaso supports a wide range of evidence sources, including:

- RAW/DD (`.dd`, `.img`, `.raw`)
- EnCase Evidence Files (`.E01`, `.Ex01`)
- VMware virtual disks (`.vmdk`)
- QCOW/QCOW2 (`.qcow`, `.qcow2`)
- AFF4 forensic images
- Physical disks and storage devices
- Mounted logical volumes and partitions

---

# Step 1 – Create the Plaso Storage File

Run the following command:

```bash
log2timeline.py --storage-file timeline.plaso image.dd
```

## Parameters

### `log2timeline.py`

The Plaso event collection tool responsible for parsing artifacts and extracting timestamped events.

### `--storage-file timeline.plaso`

Specifies the output Plaso database file.

### `image.dd`

The forensic image being analyzed.

---

# Review Timeline Statistics

To inspect the collected data:

```bash
pinfo.py timeline.plaso
```

Example output:

```text
Storage file:
  Number of events: 450,327

Parsers:
  winevtx
  winreg
  filestat
  sqlite
```

This provides a quick overview of:

- Total events
- Artifacts parsed
- Parser usage
- Collection statistics

---

# Step 2 – Export the Timeline

Once the `.plaso` file has been generated, export the data to CSV.

Run:

```bash
psort.py -o dynamic -w timeline.csv timeline.plaso
```

## Parameters

### `psort.py`

Plaso sorting and output tool.

### `-o dynamic`

Uses the Dynamic output format.

### `-w timeline.csv`

Saves the output to a CSV file.

### `timeline.plaso`

Input Plaso database.

---

# Result

A CSV timeline is created:

```text
timeline.csv
```

The exported file can be opened in:

- Microsoft Excel
- LibreOffice Calc
- [Timeline Explorer](https://ericzimmerman.github.io/#forensic-tools)
- [Timesketch](https://timesketch.org/)
- Splunk
- Elastic Stack

---

# Expected Output Files

After completing the workflow, you should have:

```text
image.dd              # Original forensic image
timeline.plaso        # Plaso timeline database
timeline.csv          # CSV timeline export
```
