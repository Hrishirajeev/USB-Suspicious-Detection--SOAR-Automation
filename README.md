# USB-Insertion SOAR Automation

Automated USB monitoring and suspicious-file investigation using **n8n, TheHive, Cortex, MISP, and VirusTotal**.

## Workflow

```text
USB Inserted
     |
     v
Read USB Serial Number
     |
     v
Check Approved USB List
     |
     +---- Approved USB ----> Monitor
     |
     +---- Unknown USB -----> Mark as Suspicious
                                   |
                                   v
                          Check USB-Related Processes
                                   |
                         +---------+---------+
                         |                   |
                    No Process          Process Found
                         |                   |
                         v                   v
                  Inspect USB Files    Collect Process Details
                  - List files         - Process name and path
                  - Check signature    - Parent process
                  - Calculate SHA-256  - Related activity
                         |                   |
                         +---------+---------+
                                   |
                                   v
                        Check Reputation and Threat Data
                         - VirusTotal
                         - MISP
                         - Cortex analyzers
                                   |
                                   v
                          Create TheHive Alert/Case
                                   |
                                   v
                            Notify the SOC
```

## Tools Used

- **n8n** — connects the tools and runs the automation.
- **TheHive** — records and manages security alerts and cases.
- **Cortex** — runs configured analyzers on supported observables.
- **MISP** — provides threat-intelligence indicators.
- **VirusTotal** — checks file hashes and reputation results.

## Investigation Logic

- Approved USB devices are monitored.
- Unknown USB devices are checked for file activity and process execution.
- File hashes, signatures, process details, and reputation results help determine risk.
- The workflow records the findings and notifies the SOC.
- High-impact actions, such as killing a process or isolating a host, require validation and safeguards.

## Lab Safety

Test with simulated events and harmless files in an isolated lab. Keep endpoint protection enabled and do not automatically perform destructive response actions without safeguards.
