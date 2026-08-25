# Fortitude RDR2 Mod Menu: Independent Security Observations

Analysis date: 2026-08-26  
Environment: Windows  
Scope: Static inspection and a five-minute launcher-only runtime observation  

## Important scope statement

This report does **not** claim that Fortitude is malware. It records reproducible observations about one locally installed build. Fortitude is a commercial game-modification and injection tool, so some behavior that resembles malware analysis indicators—packing, anti-debugging, hooks, process inspection, and DLL injection—may also serve licensing, anti-cracking, or game-modification purposes.

The injected DLL was not observed while RDR2 was running. HTTPS payloads were not decrypted. Consequently, this work cannot establish everything the protected code does or what information its servers receive.

No Fortitude binaries, licence data, crash dumps, player identifiers, usernames, or player IP addresses are included in this report.

## Samples

| Component | Version/state | Size | SHA-256 | Authenticode |
|---|---:|---:|---|---|
| `Fortitude.exe` | 1.0.49.271 | 14,461,968 bytes | `316A93730666A9AC177C24095EB9A2E1106A31CCFF992E0CDA06CEFEFB4B5E94` | Not signed |
| Injected DLL, before launcher run | Previous local copy | 32,420,880 bytes | `4788BD7536107E18077EC1E3257AF45DEB66E0CAAEBED93CF5F5098B0DF9DC4C` | Not signed |
| Injected DLL, after launcher run | Replacement local copy | 31,554,064 bytes | `732257740022CA5BE50DE6847C767EEA5172F2A4AE76274086767DF8CCE338E2` | Not signed |

The hashes identify the exact files examined; they are not, by themselves, indicators that a file is malicious.

## Confirmed static observations

- The launcher and injected DLL were not Authenticode-signed. Windows therefore could not verify a software publisher for either file.
- Both files were heavily protected with Themida-style packing/obfuscation and exhibited high-entropy content. Most original program logic, strings, and endpoints were not available for straightforward static review.
- The readable import surface included Windows HTTP functionality in the launcher and Winsock/WinINet networking capability in the DLL. These capabilities are consistent with legitimate licensing, updates, presence features, or telemetry, but could support other communication as well.
- The installation included multiple OpenSSL components, including older 1.0.2j libraries and newer 3.0.7 libraries.
- No obvious Discord-token collector, browser-cookie collector, Discord webhook, or credential-upload URL was found in the readable scripts and configuration examined. This finding does not cover the packed core.
- No Fortitude Windows service, scheduled task, or normal automatic-start entry was found. Fortitude was not running before the manual test.

## Public sandbox context

A public [Triage analysis for the launcher hash](https://tria.ge/250725-xrl2qshk5w/behavioral1) assigned a high behavioral score and reported anti-VM/anti-debugging behavior, system and process discovery, process enumeration, and Windows hook functionality. The same report recorded zero network requests during its 150-second run.

Those behaviors warrant scrutiny, but they overlap substantially with a Themida-protected game injector. The sandbox score should therefore be treated as a warning signal, not proof of credential theft or malicious intent.

## Launcher-only runtime observation

The launcher was manually opened during a five-minute metadata-only monitor. The monitor recorded Fortitude process starts, TCP connection metadata, selected file changes, and new local DNS-cache entries. It did not record keystrokes, packet payloads, credentials, or file contents.

Observed behavior:

- Two successive `Fortitude.exe` process IDs were seen, consistent with a launcher restart, handoff, or update stage.
- One established external TCP connection was observed: `104.21.28.8:443`.
- `104.21.28.8` falls within Cloudflare's published `104.16.0.0/13` IPv4 range. It is shared reverse-proxy/CDN infrastructure, so it should **not** be treated as a Fortitude-specific indicator of compromise, and the origin service cannot be identified from this IP alone.
- No additional external TCP destinations were observed during the test.
- No new DNS-cache entries were captured. The hostname may already have been cached, may have been resolved outside the observed cache behavior, or the application may have used a direct address.
- The injected DLL was replaced with the smaller, still-unsigned sample listed above.
- A small launcher state/licence file was modified. Its contents were intentionally not inspected or published.

Because port 443 traffic was encrypted and only connection metadata was collected, this test cannot determine exactly what was transmitted. Authentication, licence validation, and update checking are plausible explanations, but remain inferences.

## Local privacy observations

- Readable Fortitude logs stored other players' usernames, platform identifiers, and IP addresses in plaintext.
- The logs repeatedly recorded presence-like game location/activity information.
- A setting labelled `Enable Discord Status` was disabled, while presence-related activity still appeared in local logs. This establishes local logging, not necessarily server upload.
- Thirteen files, totalling approximately 2.6 MB, were present under a crash-log directory named `Submitted`. The directory name suggests prior submission, but the dumps were not opened because memory dumps may contain sensitive data.

No third-party player information is reproduced here.

## Local antivirus result

Local security scanning returned no harmful-item detections in the tested scope. This is useful context but not a clean bill of health: heavily packed code is inherently harder for both manual static analysis and signature-based scanners to evaluate.

## Interpretation

### Findings that reduce concern

- The launcher-only test showed a single conventional HTTPS destination rather than multiple random or residential peers.
- No persistence mechanism was found.
- No readable credential-stealing logic or obvious exfiltration endpoint was identified.
- Local security scans returned no detection.

### Findings that increase concern

- Both principal components were unsigned.
- Extensive packing and anti-analysis protection concealed the core behavior.
- The protected DLL has injection and networking capabilities and was automatically replaced by the launcher.
- Plaintext logs retained player IP addresses and identifiers.
- Submitted crash reports may contain sensitive runtime state; their exact contents and server-side retention are unknown.

## Unanswered questions for the developer

1. What fields are transmitted during launcher authentication and licence validation?
2. What information is included in submitted crash reports, where is it stored, and for how long?
3. Why are player IP addresses and identifiers retained in plaintext local logs?
4. Is presence information uploaded when Discord Status is disabled, or is it exclusively local?
5. Is there a public privacy policy covering telemetry, presence, crash reports, and retention?
6. Can the launcher and DLL be Authenticode-signed so users can verify publisher and file integrity?

## Bottom line

The observations do not demonstrate credential theft or prove that Fortitude is malware. They also do not establish that it is safe. Fortitude is high-trust software: an unsigned, heavily protected DLL is injected into a game process, while important behavior remains unavailable for ordinary inspection. A controlled in-game runtime analysis would be required to observe the injected component's actual network and filesystem behavior.

Corrections, reproducible counter-evidence, and technical responses from Fortitude's developers are welcome.

