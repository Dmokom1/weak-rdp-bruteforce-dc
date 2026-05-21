# Build Notes
# Weak RDP Exposure and Brute-Force Authentication Against a Domain Controller

This file provides supporting build context for the main README. It documents the lab setup, configuration dependencies, validation points, evidence interpretation, limitations, and screenshot mapping.

The README explains the full project story. These notes focus on what had to be configured, what was validated, and what the screenshots support.

---

## Purpose of This File

This project was built to practice controlled RDP exposure validation and weak credential testing in an isolated Active Directory lab.

These build notes focus on:

- Remote Desktop exposure
- Weak domain user creation
- Remote Desktop Users access
- Group Policy logon rights
- Policy enforcement
- RDP service validation
- Wordlist preparation
- Hydra credential testing
- Noisy authentication attempt generation
- Screenshot-supported evidence

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker Host | Kali Linux |
| Target Host | Windows Server 2022 Domain Controller |
| Domain | `cs.org` |
| Target Hostname | `WIN-HS48GJMNOGP.cs.org` |
| Target IP | `192.168.30.128` |
| Exposed Service | Remote Desktop Protocol |
| RDP Port | `3389` |
| Test User | `sally` |
| Test Password | `password` |
| Wordlist | `rockyou.txt` |
| Credential Testing Tool | Hydra |

---

## Corrected Lab Sequence

The final lab workflow followed this order:

1. Enabled Remote Desktop on the domain controller.
2. Created the weak domain user `sally`.
3. Added `sally` to Remote Desktop Users.
4. Updated Group Policy to allow logon through Remote Desktop Services.
5. Updated local logon rights needed for the lab configuration.
6. Confirmed the Default Domain Policy was linked and enforced.
7. Ran `gpupdate /force` to apply policy changes.
8. Used Nmap from Kali to confirm RDP exposure on TCP port `3389`.
9. Extracted `rockyou.txt` on Kali.
10. Used Hydra to validate the weak `sally:password` credential pair.
11. Ran a separate Administrator Hydra attempt to generate repeated authentication activity for later defender-side review.

---

## Important Values and Commands

| Item | Value |
|---|---|
| Target IP | `192.168.30.128` |
| RDP port | `3389` |
| Test user | `sally` |
| Test password | `password` |
| Domain | `cs.org` |
| RDP service shown by Nmap | `ms-wbt-server Microsoft Terminal Services` |
| Wordlist path | `/usr/share/wordlists/rockyou.txt` |
| Successful Hydra credential | `sally:password` |

### User Creation Command

```cmd
net user sally password /add /domain
```

### Group Policy Refresh Command

```cmd
gpupdate /force
```

### Rockyou Extraction Command

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

### Hydra Sally Validation Command

```bash
hydra -t 1 -V -f -l sally -P /usr/share/wordlists/rockyou.txt 192.168.30.128 rdp
```

### Hydra Administrator Attempt Command

```bash
hydra -t 1 -V -f -l administrator -P /usr/share/wordlists/rockyou.txt 192.168.30.128 rdp
```

---

## Phase 1: Remote Desktop Enabled

Remote Desktop was enabled on the domain controller.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/01-rdp-enabled-on-dc.png` | Remote Desktop enabled on the domain controller |

### Observation

The screenshot showed Remote Desktop turned on for:

`WIN-HS48GJMNOGP.cs.org`

This confirmed that the domain controller was configured to accept RDP connections in the lab.

This step created the remote access exposure needed for later validation.

---

## Phase 2: Weak Domain User Created

A weak domain user was created from an Administrator command prompt.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/02-sally-domain-user-created.png` | Weak domain user `sally` created with password `password` |

### Command Shown

```cmd
net user sally password /add /domain
```

### Observation

The command completed successfully.

This created the test account:

`cs\sally`

The password was intentionally weak:

`password`

This account was created only for controlled lab testing. It should not be treated as a safe or realistic production account configuration.

---

## Phase 3: Sally Added to Remote Desktop Users

The `sally` account was added to the Remote Desktop Users list.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/03a-sally-added-to-remote-desktop-users.png` | `sally` added to Remote Desktop Users |

### Observation

The Remote Desktop Users window showed:

`cs\sally`

This confirmed that the test account was added to the Remote Desktop Users configuration.

This was necessary, but it was not the only permission requirement. Group Policy user-right assignments also had to be configured.

---

## Phase 4: Allow Log On Through Remote Desktop Services

Group Policy was updated to allow the test user to log on through Remote Desktop Services.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/03b-allow-logon-through-rdp-policy.png` | `sally` included in RDP logon rights policy |

### Observation

The policy showed `sally` included in:

`Allow log on through Remote Desktop Services`

This confirmed that the lab account had the user-right assignment needed for RDP logon.

This was an important configuration dependency. Remote Desktop Users membership alone may not be enough if domain policy restricts who can log on through RDP.

---

## Phase 5: Allow Log On Locally

The local logon rights policy was also updated.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/03c-allow-logon-locally-policy.png` | `sally` included in local logon rights policy |

### Observation

The policy showed `sally` included in:

`Allow log on locally`

The screenshot also showed groups such as Administrators and domain-related admin groups present in the policy.

During the build, this mattered because the policy configuration had to include required built-in groups before Windows would allow the settings to be saved correctly.

This was a practical configuration lesson from the lab.

---

## Phase 6: Default Domain Policy Enforced

The Default Domain Policy was confirmed as linked and enforced.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/03d-default-domain-policy-enforced.png` | Default Domain Policy shown as linked and enforced |

### Observation

The Group Policy Management view showed the Default Domain Policy under the `cs.org` domain.

The context menu showed that the policy was linked and enforced.

This supported that the policy changes were being applied through the domain policy path used in the lab.

---

## Phase 7: Group Policy Update Applied

Group Policy was refreshed from the command line.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/04-gpupdate-force-success.png` | Group Policy update completed successfully |

### Command Shown

```cmd
gpupdate /force
```

### Observation

The command output showed:

- Computer Policy update completed successfully
- User Policy update completed successfully

This confirmed that the Group Policy changes were applied before service validation and credential testing continued.

---

## Phase 8: RDP Exposure Confirmed with Nmap

Nmap was used from Kali to confirm that RDP was reachable on the domain controller.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/05-nmap-rdp-port-open-on-dc.png` | Nmap confirmed RDP open on TCP port `3389` |

### Observation

Nmap showed:

```text
3389/tcp open ms-wbt-server Microsoft Terminal Services
```

The target was:

`192.168.30.128`

The scan also showed domain-controller-related services, including DNS, Kerberos, LDAP, SMB, and Microsoft Terminal Services.

This confirmed that the RDP service was reachable from the attacker system.

Service validation required interpretation beyond the literal service label. In this case, `ms-wbt-server` with Microsoft Terminal Services output was the indicator used to confirm exposed RDP.

---

## Phase 9: Rockyou Wordlist Prepared

The `rockyou.txt` wordlist was prepared on Kali.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/06-rockyou-wordlist-ready.png` | `rockyou.txt` extracted and ready |

### Observation

The screenshot showed that the wordlist originally existed as:

`rockyou.txt.gz`

It was then decompressed into:

`/usr/share/wordlists/rockyou.txt`

This confirmed that the wordlist was ready before Hydra testing.

This was a practical setup issue. Hydra could not use the compressed `.gz` file directly in the intended way, so the wordlist had to be extracted first.

---

## Phase 10: Hydra Validated the Weak Sally Credentials

Hydra was used to test the weak `sally` credentials against RDP.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/07-hydra-sally-success.png` | Hydra validated `sally:password` against RDP |

### Observation

Hydra found the valid credential pair:

```text
login: sally
password: password
```

The target service was:

```text
rdp://192.168.30.128:3389
```

This confirmed that the intentionally weak domain account could be validated through the exposed RDP service.

This is the strongest validation point in the project.

It proves weak credential validation against exposed RDP in the lab. It does not prove full compromise, post-login activity, or domain escalation.

---

## Phase 11: Administrator Attempt Generated Noisy Activity

A separate Hydra attempt was launched against the Administrator account.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `screenshots/08-hydra-administrator-attempt.png` | Administrator Hydra attempt generated repeated authentication activity |

### Observation

Hydra was run against:

`administrator`

on:

`192.168.30.128`

The screenshot showed repeated attempts over time before the session was manually stopped.

This phase was not intended to show Administrator compromise. Its purpose was to generate noisy authentication activity that could later be reviewed from the defender side.

---

## Evidence Interpretation Notes

These notes keep the project explanation accurate:

- Remote Desktop was intentionally enabled for the lab.
- The weak user `sally` was intentionally created for controlled testing.
- Remote Desktop Users membership alone was not the full requirement.
- Group Policy logon rights had to be configured for the intended behavior.
- `gpupdate /force` confirmed policy application.
- Nmap confirmed RDP exposure through TCP port `3389`.
- `rockyou.txt` had to be decompressed before use.
- Hydra validated the weak `sally:password` credential pair.
- The Administrator Hydra attempt generated repeated authentication activity but did not show compromise.
- This project focuses on exposure and authentication validation, not defender-side investigation.
- Defender-side alert review belongs in the separate RDP investigation project.

---

## Key Lessons Learned

1. RDP exposure on a domain controller creates a risky authentication surface.
2. Weak passwords can turn remote access exposure into a practical access path.
3. Group membership and user-right assignments both matter for RDP access.
4. Service validation should happen before credential testing.
5. Nmap’s service labels require interpretation, especially for RDP.
6. Wordlist preparation matters before password testing.
7. A successful Hydra result proves credential validation, not full compromise.
8. Failed or interrupted Hydra activity can still be useful for defender-side telemetry review.
9. Long-running password-testing activity is more reliable when the attacker VM is configured not to sleep or lock unexpectedly.
10. Public documentation should clearly separate controlled lab validation from real-world compromise claims.

---

## Improvements for a Future Version

Future improvements could include:

- Capturing Windows Security Event IDs for successful and failed logons.
- Capturing endpoint telemetry from the domain controller during Hydra testing.
- Showing whether `sally` completed an interactive RDP session.
- Documenting account lockout policy behavior.
- Testing the same scenario with a stronger password policy.
- Comparing Remote Desktop Users membership with Group Policy user-right assignments.
- Capturing Security Onion or Elastic alerts from the same test window.
- Adding a timeline from configuration changes to Nmap validation to Hydra testing.
- Documenting Kali power and screen-lock settings if long-running scans are used.

---

## Screenshot Map

| Screenshot | What It Supports |
|---|---|
| `screenshots/01-rdp-enabled-on-dc.png` | Remote Desktop enabled on the domain controller |
| `screenshots/02-sally-domain-user-created.png` | Weak domain user `sally` created with password `password` |
| `screenshots/03a-sally-added-to-remote-desktop-users.png` | `sally` added to Remote Desktop Users |
| `screenshots/03b-allow-logon-through-rdp-policy.png` | `sally` included in RDP logon rights policy |
| `screenshots/03c-allow-logon-locally-policy.png` | `sally` included in local logon rights policy |
| `screenshots/03d-default-domain-policy-enforced.png` | Default Domain Policy shown as linked and enforced |
| `screenshots/04-gpupdate-force-success.png` | Group Policy update completed successfully |
| `screenshots/05-nmap-rdp-port-open-on-dc.png` | Nmap confirmed RDP open on TCP port `3389` |
| `screenshots/06-rockyou-wordlist-ready.png` | `rockyou.txt` extracted and ready |
| `screenshots/07-hydra-sally-success.png` | Hydra validated `sally:password` against RDP |
| `screenshots/08-hydra-administrator-attempt.png` | Administrator Hydra attempt generated repeated authentication activity |