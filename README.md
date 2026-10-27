# USB-Insertion-SOAR

Automated USB-device checks, file and process investigation, threat-intelligence enrichment, and incident case creation using **n8n, TheHive, Cortex, MISP, and VirusTotal**.

---

## What this is

A lab-built SOAR workflow for investigating USB devices connected to a Windows endpoint. It checks whether a device is approved, looks for activity launched from the USB, gathers evidence, enriches file hashes, and records the investigation for the SOC.

- Extracts the USB serial number and compares it with an approved-device list
- Checks for processes launched from the USB drive after insertion
- Collects file hashes, process details, and available signature information
- Checks hashes against VirusTotal and indicators against MISP
- Uses configured Cortex analyzers for additional analysis
- Creates an alert or case in TheHive and notifies the SOC

The workflow supports analyst decisions; response actions such as terminating a process or isolating an endpoint need validation and suitable safeguards.

---

## Architecture

```text
USB connected to a Windows endpoint
(or an IoT maintenance workstation / supported gateway)
                         |
                         v
             Endpoint event collector
     USB serial, file details, process activity
                         |
                         v
                  n8n Webhook
                         |
                         v
              Check approved USB list
                    /         \
                 Match       No match
                   |             |
                   v             v
              Monitor only   Mark suspicious
                                 |
                                 v
                   Check process activity (0–2 min)
                                 |
                    +------------+------------+
                    |                         |
                 No execution              Execution
                    |                         |
                    v                         v
            Passive file checks       Collect process evidence
            - List files              - Process name and path
            - SHA-256 hash            - Parent process
            - Signature               - File hash
                    |                         |
                    +------------+------------+
                                 |
                                 v
                    VirusTotal / MISP lookups
                                 |
                                 v
                      Cortex analysis (optional)
                                 |
                                 v
                       Create/update TheHive case
                                 |
                                 v
                          Notify the SOC
```

The Windows endpoint or supported gateway must send the USB and process events to n8n. The webhook alone does not detect USB insertion.

---

## Tools and their roles

| Tool | Role |
| --- | --- |
| **Windows endpoint / event collector** | Detects USB activity and supplies file and process details. |
| **n8n** | Connects the steps and runs the automation workflow. |
| **VirusTotal** | Checks file-hash reputation using available threat-intelligence results. |
| **MISP** | Looks up known indicators in threat-intelligence data. |
| **Cortex** | Runs configured analyzers against supported observables. |
| **TheHive** | Stores alerts/cases and tracks investigation evidence and status. |

---

## If EDR already exists, why use this?

**This workflow is not a replacement for EDR.** EDR is designed to monitor endpoint activity and detect or respond to threats. Depending on the product, licensing, configuration, and device support, an EDR may already detect USB activity or suspicious process execution.

This SOAR workflow demonstrates how to coordinate a specific removable-media investigation across multiple tools:

- Compare USB serial numbers with an approved-device list
- Bring USB insertion and process-execution evidence into one investigation
- Enrich file hashes with threat-intelligence results
- Create or update a documented TheHive case and notify the SOC
- Apply consistent decision logic instead of manually repeating the same steps

An existing EDR can be the event source for this workflow. If the EDR already performs these checks and integrations well, this workflow may be unnecessary for that environment. Its value is the cross-tool automation, repeatable investigation, and learning of SOAR integration—not replacing endpoint protection.

---

## Why IoT is relevant

USB drives are sometimes used by engineers or technicians to transfer firmware, configuration files, logs, or updates to **IoT devices and their gateways**. These devices may have limited support for traditional endpoint agents, including EDR.

This workflow can help protect that transfer path by monitoring the Windows maintenance workstation or a supported gateway used to service IoT equipment. In an IoT scenario, the investigation can include the target device or gateway ID, the USB serial number, the firmware/configuration file hash, and whether the file matches an approved release.

**Important:** This does not mean n8n can automatically see USB activity on every IoT device. A compatible endpoint sensor, gateway, or other event source must collect the relevant events and send them to n8n. Sensitive firmware should not be uploaded to public analysis services; use an approved hash lookup or an internal analyzer according to the organization's policy.

---

## How it decides

**1. Is the USB approved?**  
Compare its serial number with the approved-device list. An approved device is monitored; an unknown device is marked for investigation.

**2. Did a process run from the USB?**  
Check process activity during the configured observation window after insertion. Record the process name, executable path, parent process, and related file hash when available.

**3. What is on the drive?**  
If no execution is detected, list files, calculate SHA-256 hashes, check available digital signatures, and perform reputation lookups. For IoT maintenance use cases, compare firmware or configuration hashes with an approved release list where available.

**4. What does the evidence say?**  
Use reputation results and behavior indicators to support triage. A clean reputation result alone does not prove a file is safe, and a single signal should not automatically determine severity.

**5. Record and notify.**  
Send the relevant evidence and findings to TheHive and notify the SOC so an analyst can decide the next action.

---

## Response safeguards

Terminating processes, quarantining files, isolating endpoints, or disabling user accounts can disrupt legitimate work. Keep these actions disabled by default in the lab; enable them only with tested conditions, appropriate permissions, and analyst approval where required. Start with evidence collection, case creation, and notification.

---

## Setup requirements

- A Windows test VM with USB access configured
- An endpoint event source that can report USB serial numbers and process/file details
- An Ubuntu host running n8n
- Reachable TheHive, Cortex, and MISP services
- API credentials for the services used in the workflow

Configure credentials inside n8n. Do not commit API keys, passwords, tokens, or sensitive endpoint data to GitHub.

---

## Repository contents

```text
├── workflow.json     # Exported n8n workflow, if included
├── README.md
└── screenshots/      # Workflow and execution screenshots, if included
```

---

## Disclaimer

Built for authorized lab learning and testing. Use controlled test data, respect organizational data-handling policies, and do not run response actions against systems without authorization.
