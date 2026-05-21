# Weak RDP Exposure and Brute-Force Authentication Against a Domain Controller

This project was completed in an isolated Active Directory lab built for controlled exposure validation and authentication testing.

---

## Overview

This project demonstrates how exposed Remote Desktop Protocol, weak password selection, and permissive logon rights can create a risky authentication path to a domain controller.

In the lab, I enabled Remote Desktop on a Windows Server 2022 domain controller, created a deliberately weak domain user, configured the logon rights required for remote access, confirmed that TCP port `3389` was reachable from Kali Linux, and used Hydra to validate the weak credential exposure.

The goal was not to claim a production compromise. The goal was to build a controlled authentication-abuse scenario that could later produce useful telemetry for defender-side investigation in Security Onion and Elastic.

---

## Why I Built This

RDP is a common remote administration service, but exposing it with weak credentials creates a serious security risk.

I built this lab to practice:

1. Understanding how RDP exposure changes the attack surface of a domain controller.
2. Configuring the Windows permissions required for a test user to authenticate through RDP.
3. Validating service exposure from an attacker system.
4. Preparing a password wordlist for testing.
5. Using Hydra to validate weak credential risk in a controlled lab.
6. Generating authentication activity that could later be reviewed from the defender’s point of view.

This project is best understood as the setup and validation phase for later detection and investigation work.

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
| Tool Used for Credential Testing | Hydra |

---

## Tools Used

| Tool | Purpose |
|---|---|
| Windows Remote Desktop Settings | Enabled RDP on the domain controller |
| Command Prompt | Created the test domain user |
| Group Policy Management | Configured logon rights |
| `gpupdate /force` | Applied policy changes |
| Nmap | Confirmed RDP service exposure |
| Hydra | Tested weak credentials against RDP |
| Kali Linux | Attacker/test system |
| `rockyou.txt` | Password wordlist used for testing |

---

## Project Flow

The project followed this sequence:

1. Enabled Remote Desktop on the domain controller.
2. Created a weak domain user named `sally`.
3. Added `sally` to Remote Desktop Users.
4. Updated Group Policy to allow RDP logon.
5. Updated local logon rights needed for the lab configuration.
6. Enforced the Default Domain Policy.
7. Ran `gpupdate /force`.
8. Confirmed TCP port `3389` was open from Kali.
9. Extracted and validated the `rockyou.txt` wordlist.
10. Used Hydra to validate the weak `sally` credentials.
11. Ran a separate Administrator attempt to generate additional authentication activity.

---

## Phase 1: Remote Desktop Enabled

Remote Desktop was enabled on the domain controller.

![RDP Enabled on DC](screenshots/01-rdp-enabled-on-dc.png)

### What this proved

The screenshot showed Remote Desktop turned on for:

`WIN-HS48GJMNOGP.cs.org`

This confirmed that the domain controller was configured to accept RDP connections in the lab.

Enabling RDP increased the attack surface and created the service exposure needed for later credential testing.

---

## Phase 2: Weak Domain User Created

A weak domain user was created from an Administrator command prompt.

![Sally Domain User Created](screenshots/02-sally-do
main-user-created.png)

### Command Shown

```cmd
net user sally password /add /domain
```

### What this proved

The command completed successfully, creating the domain user:

`cs\sally`

The password was intentionally weak:

`password`

This user was created only for controlled lab testing. The weak credential was intentionally chosen to demonstrate how poor password hygiene can turn remote access exposure into a practical authentication risk.

---

## Phase 3: Sally Added to Remote Desktop Users

The `sally` account was added to the Remote Desktop Users list.

![Sally Added to Remote Desktop Users](screenshots/03a-sally-added-to-remote-desktop-users.png)

### What this proved

The screenshot showed:

`cs\sally`

listed under Remote Desktop Users.

This confirmed that the test account was allowed through the Remote Desktop Users configuration.

This step alone was not enough for the full scenario. Group Policy logon rights also had to be adjusted.

---

## Phase 4: Allow Log On Through Remote Desktop Services

Group Policy was updated to allow the relevant users and groups to log on through Remote Desktop Services.

![Allow Logon Through RDP Policy](screenshots/03b-allow-logon-through-rdp-policy.png)

### What this proved

The policy view showed `sally` included in:

`Allow log on through Remote Desktop Services`

This confirmed that the test user had the user-right assignment needed for RDP logon in the lab.

This was an important configuration dependency. Adding a user to Remote Desktop Users alone does not always make the scenario work if user-right assignments are restrictive.

---

## Phase 5: Allow Log On Locally

The local logon rights policy was also updated.

![Allow Logon Locally Policy](screenshots/03c-allow-logon-locally-policy.png)

### What this proved

The policy view showed `sally` included in:

`Allow log on locally`

The screenshot also showed other required groups such as Administrators and domain-related groups present in the policy.

This supported the lab configuration needed for the account to authenticate successfully in the controlled environment.

---

## Phase 6: Default Domain Policy Enforced

The Default Domain Policy was confirmed as linked and enforced.

![Default Domain Policy Enforced](screenshots/03d-default-domain-policy-enforced.png)

### What this proved

The Group Policy Management view showed the Default Domain Policy in the `cs.org` domain with enforcement visible.

This supported that the policy changes were being applied through the domain policy path used in the lab.

---

## Phase 7: Group Policy Update Applied

Group Policy was refreshed from the command line.

![Group Policy Update Success](screenshots/04-gpupdate-force-success.png)

### Command Shown

```cmd
gpupdate /force
```

### What this proved

The command output showed:

- Computer Policy update completed successfully
- User Policy update completed successfully

This confirmed that the policy changes were applied successfully before credential testing continued.

---

## Phase 8: RDP Exposure Confirmed with Nmap

Nmap was used from Kali to confirm that RDP was reachable on the domain controller.

![Nmap RDP Port Open on DC](screenshots/05-nmap-rdp-port-open-on-dc.png)

### What this proved

The scan showed:

```text
3389/tcp open ms-wbt-server Microsoft Terminal Services
```

The target was:

`192.168.30.128`

The scan also showed domain-related service information, including:

- DNS
- Kerberos
- LDAP
- SMB
- Microsoft Terminal Services

This confirmed that the domain controller was reachable from Kali and that RDP was exposed on TCP port `3389`.

---

## Phase 9: Rockyou Wordlist Prepared

The `rockyou.txt` wordlist was prepared on Kali.

![Rockyou Wordlist Ready](screenshots/06-rockyou-wordlist-ready.png)

### What this proved

The screenshot showed that `rockyou.txt.gz` existed first, then was decompressed into:

`/usr/share/wordlists/rockyou.txt`

This confirmed that the password list was available before running Hydra.

This was a practical setup issue. The wordlist had to be extracted before it could be used.

---

## Phase 10: Hydra Validated the Weak Sally Credentials

Hydra was used to test the weak `sally` account against RDP.

![Hydra Sally Success](screenshots/07-hydra-sally-success.png)

### What this proved

Hydra found a valid credential pair:

```text
login: sally
password: password
```

The target service was:

```text
rdp://192.168.30.128:3389
```

This confirmed that the intentionally weak domain account could be validated through the exposed RDP service.

This is the strongest evidence in the project. It proves weak credential validation against exposed RDP in the lab. It should not be overstated as full system compromise or post-login activity.

---

## Phase 11: Administrator Attempt Generated Noisy Activity

A separate Hydra attempt was launched against the Administrator account.

![Hydra Administrator Attempt](screenshots/08-hydra-administrator-attempt.png)

### What this proved

The screenshot showed Hydra running against:

`administrator`

on:

`192.168.30.128`

The attempt generated repeated authentication attempts and was manually stopped.

This phase was not evidence of Administrator compromise. Its purpose was to create noisy authentication activity that could later be reviewed from the defender side.

---

## Key Findings

### 1. RDP was exposed on the domain controller

Nmap confirmed TCP port `3389` was open and associated with Microsoft Terminal Services.

### 2. A weak domain user was intentionally created

The `sally` account was created with the weak password `password` for controlled testing.

### 3. Logon rights mattered

Remote Desktop Users membership alone was not the full configuration. Group Policy logon rights had to be adjusted for the scenario to work.

### 4. Hydra validated the weak credential risk

Hydra successfully found the `sally:password` credential pair against the exposed RDP service.

### 5. The Administrator attempt was activity generation, not compromise

The Administrator Hydra run generated repeated attempts but should not be described as successful compromise.

---

## Detection and Lab Value

This project created the authentication activity needed for later defender-side analysis.

From a SOC perspective, the value is in understanding how the following conditions combine:

- Exposed remote access
- Weak password selection
- Permissive logon rights
- Repeated authentication attempts
- Service validation from an attacker system

The project supports later investigation work in Security Onion and Elastic, where the resulting RDP and authentication-related activity can be reviewed.

---

## Limitations

This was a controlled lab, not a production environment.

Important limitations:

- The weak account was intentionally created for testing.
- The password was deliberately insecure.
- The screenshots prove credential validation, not full interactive login activity.
- The Administrator attempt does not show compromise.
- No endpoint or Windows Event Log screenshots are included in this project.
- Defender-side investigation is handled in a separate project.
- A production environment should not expose domain controller RDP broadly or use weak credentials.
- A full investigation would require Windows Security logs, endpoint telemetry, network detections, and authentication event review.

---

## Improvements for a Future Version

If I expanded this project, I would improve it by:

- Capturing Windows Security Event IDs from the domain controller.
- Showing successful and failed logon events tied to the test activity.
- Capturing Elastic or Security Onion telemetry from the same time window.
- Documenting whether the `sally` account could complete an interactive RDP session.
- Adding firewall or network-level evidence for the RDP traffic.
- Testing account lockout policy behavior.
- Comparing a weak account test with a properly protected account.
- Adding a short timeline from configuration to scan to Hydra validation.

---

## Screenshot Evidence

| Screenshot | What It Shows |
|---|---|
| `screenshots/01-rdp-enabled-on-dc.png` | Remote Desktop enabled on the domain controller |
| `screenshots/02-sally-domain-user-created.png` | Weak domain user `sally` created with password `password` |
| `screenshots/03a-sally-added-to-remote-desktop-users.png` | `sally` added to Remote Desktop Users |
| `screenshots/03b-allow-logon-through-rdp-policy.png` | `sally` included in RDP logon rights policy |
| `screenshots/03c-allow-logon-locally-policy.png` | `sally` included in local logon rights policy |
| `screenshots/03d-default-domain-policy-enforced.png` | Default Domain Policy shown as linked/enforced |
| `screenshots/04-gpupdate-force-success.png` | Group Policy update completed successfully |
| `screenshots/05-nmap-rdp-port-open-on-dc.png` | Nmap confirmed RDP open on TCP port `3389` |
| `screenshots/06-rockyou-wordlist-ready.png` | `rockyou.txt` extracted and ready |
| `screenshots/07-hydra-sally-success.png` | Hydra validated `sally:password` against RDP |
| `screenshots/08-hydra-administrator-attempt.png` | Administrator Hydra attempt generated repeated authentication activity |

---

## Conclusion

This project validated how a weak domain account and exposed RDP service can create a dangerous authentication path in an Active Directory lab.

The most important result was not just that Hydra found the weak `sally` credentials. The value was understanding the configuration chain that made the scenario possible: RDP exposure, account creation, Remote Desktop Users access, logon rights, policy enforcement, service reachability, and credential testing.

This project is best treated as a controlled exposure and authentication validation lab that supports later defender-side analysis.