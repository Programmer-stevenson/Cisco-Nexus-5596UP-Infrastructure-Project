
<div align="center">
  <img src="assets/brandons-logo.png" alt="Brandon Stevenson — IT Professional" width="180">

# Cisco Nexus 5596UP — TFTP-Based NX-OS Recovery, Secure Data Sanitization & Infrastructure Validation

**Physical Cisco Nexus N5K-C5596UP | Cisco NX-OS | TFTP Recovery | Bootflash | Hardware Diagnostics**

**[View Technical Project Report (PDF)](./Cisco_Nexus_5596UP_Technical_Project_Reviewed.pdf)**
</div>

<a id="table-of-contents"></a>
## Table of Contents

1. [Project Overview](#project-overview)
2. [Hardware and Tools](#hardware-and-tools)
3. [Physical Equipment](#physical-equipment)
4. [Bootloader Access and TFTP Recovery](#bootloader-access-and-tftp-recovery)
5. [NX-OS Image Transfer and Boot Configuration](#nx-os-image-transfer-and-boot-configuration)
6. [NX-OS and Hardware Verification](#nx-os-and-hardware-verification)
7. [System and Interface Diagnostics](#system-and-interface-diagnostics)
8. [Configuration and System Initialization](#configuration-and-system-initialization)
9. [Project Results](#project-results)
10. [Technical Report](#technical-report)
11. [About](#about)

---

<a id="project-overview"></a>
## 1. Project Overview

This project documents hands-on recovery and servicing of a **physical Cisco Nexus N5K-C5596UP data-center switch**. Using a serial-console connection and TFTP server, I accessed the bootloader, recovered Cisco NX-OS, transferred operating-system images to internal bootflash, worked with boot settings, and inspected the switch through the NX-OS CLI.

I also performed hardware and system diagnostics, reviewed interface status and configuration, and completed system-initialization tasks. The walkthrough follows the actual terminal output, including commands that returned errors or compatibility warnings.

The goal is to show the practical process of working with real enterprise networking hardware—not a simulation.

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="hardware-and-tools"></a>
## 2. Hardware and Tools

| Component | Purpose |
|---|---|
| Cisco Nexus **N5K-C5596UP** | Physical enterprise data-center switch |
| **Tera Term** and serial-console cable | Direct access to bootloader and NX-OS CLI |
| **TFTP server** | Network-based delivery of recovery images |
| Cisco **NX-OS kickstart and system images** | Boot and restore the operating system |
| Internal **bootflash** | Store operating-system image files |
| Built-in NX-OS diagnostics | Inspect modules, power, interfaces, and system resources |

The switch and TFTP server communicated on the same local subnet; a default gateway was not needed for the transfer.

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="physical-equipment"></a>
## 3. Physical Equipment

**Nexus 5596UP — chassis and front interfaces**

![Cisco Nexus 5596UP chassis](assets/nk5000full.png)

**Internal chassis — hardware layout**

![Interior of Cisco Nexus chassis](assets/Cisco_Internal_Chassis_Cropped.jpg)

**Internal circuit board — close-up**

![Cisco Nexus internal PCB](assets/Cisco_Internal_PCB_Cropped.jpg)

These photographs show the actual chassis and internal components associated with the project.

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="bootloader-access-and-tftp-recovery"></a>
## 4. Bootloader Access and TFTP Recovery

The switch was accessed through its serial console. From the `loader>` environment, I checked its network settings and assigned a temporary management IP address so it could communicate directly with the TFTP server.

**Key commands:** `show ip`, `set ip`, and `boot tftp://...`.

- `show ip` displays the bootloader's network settings.
- `set ip` supplies the switch with an address and subnet mask for the recovery connection.
- `boot tftp://...` retrieves and starts a compatible kickstart image from the network server.

**Terminal evidence**

![Bootloader and TFTP recovery](evidence/nexus500-step1.png)

Next, `write erase` cleared the saved startup configuration and `reload` restarted the switch during the recovery workflow.

![Kickstart boot and configuration reset](evidence/nexus500-step2.png)

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="nx-os-image-transfer-and-boot-configuration"></a>
## 5. NX-OS Image Transfer and Boot Configuration

After kickstart loaded, I configured the management interface and copied the NX-OS system image from the TFTP server into **bootflash**, the switch's local image storage.

**Key commands:** `interface mgmt0` / `interface m0`, `ip address`, `no shutdown`, `copy tftp: bootflash:`, and `dir bootflash:`.

- The management-interface commands establish connectivity to the TFTP server.
- `copy tftp: bootflash:` saves the network-hosted image locally.
- `dir bootflash:` confirms the image files are present.

**Image transfer to bootflash**

![Copy NX-OS image from TFTP to flash](evidence/nexus500-step%20after%20xopying%20flash.png)

**Bootflash inspection immediately after the transfer**

![Inspect image files after copying to bootflash](evidence/nexus500-step%20after%20step%20of%20copying%20flash.png)

**Boot-variable configuration and verification**

The `boot system bootflash:...` command selects the NX-OS system image for startup. `copy run start` saves the active settings, while `show boot` displays the configured kickstart and system paths.

![Boot-variable configuration](evidence/nexus500-step3.png)

![Review kickstart and system boot variables](evidence/nexus500-step4.png)

The output includes an **image compatibility warning**. Kickstart and system image releases must be compatible; the warning is recorded rather than treated as a successful compatibility check.

**Review local images and load the system image**

`dir bootflash:` checks the stored image filenames. `load bootflash:...` starts the selected NX-OS system image from local storage.

![Local bootflash image listing](evidence/nexus500-step5.png)

![Load NX-OS system image](evidence/nexus500-step6.png)

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="nx-os-and-hardware-verification"></a>
## 6. NX-OS and Hardware Verification

Once NX-OS was running, I checked the operating-system release, switch model, installed modules, and boot paths.

**Key commands:** `terminal length 0`, `show version`, `show module`, and `show boot`.

- `terminal length 0` displays longer command results without paging.
- `show version` identifies the NX-OS release and chassis.
- `show module` lists installed hardware modules and their status.
- `show boot` shows the image paths configured for startup.

![NX-OS version and chassis identification](evidence/nexus500-step6.png)

![Module and boot-path inspection](evidence/nexus500-step7.png)

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="system-and-interface-diagnostics"></a>
## 7. System and Interface Diagnostics

I ran diagnostic commands to inspect hardware condition, operating-system resources, and Ethernet interfaces.

**Power and module diagnostics**

- `show environment power` reports the status of installed power supplies. The captured result shows one supply operating and another reporting a failure/shutdown state.
- `show diagnostic result module all` displays the switch's built-in module test results. The recorded module test output reports **PASS**.

![Power supply status](evidence/nexus500-step8.png)

![Module diagnostic results](evidence/nexus500-step9.png)

**Operating-system resources**

`show system resources` reports CPU and memory usage. `show processes cpu` provides a more detailed view of running processes. An initial `show process cpu` attempt was rejected, and the command was corrected.

![CPU and memory monitoring](evidence/nexus500-step10.png)

**Ethernet interface checks**

`show interface` displays a port's state, speed, counters, and errors. `show interface brief` gives a broader view across interfaces, including reasons for down states such as no cable or missing transceiver.

![Individual Ethernet interface inspection](evidence/nexus500-step11.png)

![Ethernet interface status summary](evidence/nexus500-step12.png)

**Inventory and management state**

`show inventory` / `show inv` identifies the installed chassis and components. Interface output also shows management addressing and link status.

![Management interface and hardware inventory](evidence/nexus500-step13.png)

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="configuration-and-system-initialization"></a>
## 8. Configuration and System Initialization

Before initialization, I inspected active and saved configurations along with the switch's Layer 2 state.

**Key commands:** `show running-config`, `show startup-config`, `show mac address-table`, `show spanning-tree summary`, and `show diagnostic bootup level`.

- The running and startup configuration commands show current settings and what is saved for reboot.
- The MAC address table shows learned Layer 2 addresses when present.
- Spanning Tree output shows loop-prevention settings.
- Startup diagnostics show the configured boot-time test level.

A MAC-table command initially returned a syntax error; the corrected form was used in the terminal session.

![Configuration and Layer 2 inspection](evidence/nexus500-step14.png)

![Startup diagnostic level](evidence/nexus500-step15.png)

**System initialization**

`init system` begins system initialization, removing stored configuration, licenses, and bootflash contents after confirmation. The terminal output records the filesystem preparation and initialization sequence.

![NX-OS system initialization](evidence/nexus500-step16.png)

**Final boot settings and storage review**

`write erase boot` clears stored boot variables. `write erase debug` and `write erase` clear additional saved diagnostic/configuration data. `show ip` and directory commands inspect the resulting management and storage state. The output also records unsupported or mistyped commands, including `clear nvram`, which is not supported in this Nexus 5500 environment.

![Boot-setting cleanup and post-initialization inspection](evidence/nexus500-step17.png)

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="project-results"></a>
## 9. Project Results

This project documents a full hands-on sequence across **bootloader access, TFTP-based kickstart recovery, local NX-OS image transfer, boot configuration, hardware diagnostics, interface inspection, and system initialization**.

The terminal record also captures real troubleshooting points, including image-pair compatibility warnings and command corrections. Those details are part of working through the actual device behavior, rather than a simulated or idealized command sequence.

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="technical-report"></a>
## 10. Technical Report

The report provides the complete phase-by-phase walkthrough with terminal evidence and explanations of each command.

**[View the Cisco Nexus 5596UP Technical Project Report (PDF)](./Cisco_Nexus_5596UP_Technical_Project_Reviewed.pdf)**

[↑ Back to Table of Contents](#table-of-contents)

---

<a id="about"></a>
## 11. About

I'm Brandon Stevenson, an IT Infrastructure Technician working with enterprise server and network equipment.

My experience includes physical infrastructure servicing, Cisco networking equipment, server management interfaces, operating-system imaging, hardware troubleshooting, and technical documentation.

This repository documents one of my hands-on infrastructure projects using physical enterprise networking hardware.

My technical projects reflect this combination of hands-on enterprise hardware experience, IT administration, programming, and infrastructure automation.

**Portfolio:** [brandons-resume.com](https://brandons-resume.com)  
**GitHub:** [Programmer-stevenson](https://github.com/Programmer-stevenson)  
**LinkedIn:** [Brandon Stevenson](https://www.linkedin.com/in/brandon-in-tech/)

[↑ Back to Table of Contents](#table-of-contents)
