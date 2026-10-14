# VirtualBox — Lab Environment

## 1. Purpose
VirtualBox runs guest operating systems as virtual machines (VMs) on your Windows host. In this lab it can run Kali Linux and Metasploitable 2 separately.

**Memory hook:** VirtualBox = **the classroom**; VMs = the students.

## 2. Inputs
- VM image: usually an `.iso`, `.ova`, or virtual disk such as `.vdi`.
- RAM and CPU allocation.
- Virtual disk size.
- Network mode: NAT, Host-only, Bridged, or Internal Network.

## 3. GUI workflow
1. Open VirtualBox → **New** to create a VM, or **Import Appliance** for an OVA.
2. Select the OS type and version.
3. Allocate reasonable RAM/CPU without starving Windows.
4. Attach the ISO or existing virtual disk.
5. Open **Settings → Network**.
6. For this lab, a common safe design is Kali Adapter 1 = NAT for updates and Adapter 2 = Host-only for lab traffic; Metasploitable 2 = Host-only only. Ensure both lab adapters use the same Host-only network.
7. Start the VM and confirm it boots.

## 4. Useful checks
There is no single VirtualBox terminal command required for normal GUI use. On Windows, `VBoxManage` may be available from the VirtualBox installation directory:
```powershell
VBoxManage list vms
VBoxManage list runningvms
VBoxManage showvminfo "Kali"
```
If PowerShell says the command is not found, use the full path to `VBoxManage.exe` or use the GUI.

## 5. Output and interpretation
- `list vms`: registered VM names and IDs.
- `list runningvms`: only VMs currently running.
- `showvminfo`: configuration details such as memory, storage, and networking.
- In the GUI, a running VM window shows the guest OS; a network adapter setting shows how it connects.

**Say in viva:** “VirtualBox provides an isolated virtual lab. Host-only networking lets the lab VMs communicate with each other without making the vulnerable target directly reachable from the physical LAN.”

## 6. Common problems
- **No IP / cannot ping:** check both VMs use the same Host-only network and are powered on.
- **Target exposed:** shut it down and remove Bridged networking/port forwarding.
- **VM slow:** reduce simultaneous VMs or adjust RAM/CPU carefully.
- **No 64-bit guest option:** check hardware virtualization is enabled in firmware and not blocked by host configuration.

## 7. Viva questions
**Q: NAT vs Host-only?** NAT usually gives a VM outbound connectivity through the host; Host-only creates a private network between host and participating VMs.
**Q: Why not Bridged for Metasploitable 2?** It can expose a deliberately vulnerable OS to the physical network.
**Q: What is a snapshot?** A point-in-time VM state that can help return a lab to a known condition; it is not a substitute for backups.
**Q: Host vs guest?** Host is the physical computer; guest is the OS running inside the VM.

## 8. One-line output explanation
“The VM is running with **[network mode]**, so it can communicate with **[intended peers]**; I verify isolation and connectivity before testing.”
