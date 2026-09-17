# How to Install Wireshark on Ubuntu

This tutorial explains how to install and launch the latest stable version of Wireshark on Ubuntu using the official PPA.

---

### Step 1: Add the Official Wireshark PPA

Open your terminal and run the following command to add the official PPA repository:

```bash
sudo add-apt-repository ppa:wireshark-dev/stable
```

> **Note:** When prompted, press <kbd>Enter</kbd> to confirm and proceed with adding the repository.

---

### Step 2: Update Package Lists & Install Wireshark

Run the update command, followed by the installation command:

```bash
sudo apt update
sudo apt install wireshark
```

---

### Step 3: Configure Non-Superuser Packet Capture

During the installation process, a configuration screen will appear titled **Configuring wireshark-common**:

![Configuring wireshark-common](Screenshot%20from%202026-09-17%2013-58-32.png)

1. Use the directional arrow keys (<kbd>←</kbd> / <kbd>→</kbd>) to highlight **`<Yes>`**.
2. Press <kbd>Enter</kbd> to confirm.

---

### Step 4: Launch Wireshark

You can launch Wireshark using either of the following methods:

- **Via Terminal:**
  ```bash
  wireshark
  ```
- **Via Application Menu:** Open your desktop application menu, search for **Wireshark**, and click the icon to launch.
