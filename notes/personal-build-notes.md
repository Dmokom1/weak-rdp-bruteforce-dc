# Lab Implementation Notes

## Key Observations from the Build

- Remote Desktop Users membership alone was not sufficient for the scenario; the relevant user rights assignments also had to be updated through Group Policy to allow the intended logon behavior.
- The local logon policy required the built-in `Administrators` group to be explicitly present before Windows would allow the configuration to be saved.
- Service validation required interpretation beyond a literal protocol label. Nmap identified `3389/tcp open ms-wbt-server` with Microsoft Terminal Services output, which was the indicator used to confirm exposed RDP.
- The `rockyou.txt` wordlist was initially still compressed on Kali, so `rockyou.txt.gz` had to be extracted before it could be used with Hydra.
- The Hydra run against `administrator` was intentionally allowed to generate visible repeated attempts and then stopped once enough brute-force-style activity had been produced for later review.
- Long-running scans and password-testing activity were more reliable after adjusting Kali’s sleep and lock behavior to prevent interruptions during execution.

## Why These Notes Matter

These notes capture the implementation details that affected how the scenario behaved in practice. They reflect configuration dependencies, service interpretation, and execution reliability rather than just the attack steps themselves.

That matters because the value of the lab was not only in creating a weak access path, but also in understanding the conditions required to make the scenario work correctly and produce useful authentication activity for later defender-side analysis.