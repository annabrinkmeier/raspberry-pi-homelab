# Audit finding: my "private" DNS resolver was forwarding to a public resolver

**Date found:** November 15, 2026
**System:** Pi 2 (Pi-hole + Unbound)
**Impact:** Privacy. DNS kept working, so nothing looked wrong.
**Status:** Fixed and verified. One follow-up test (a planned reboot) is still open.

## Summary

I set up Unbound so my network would resolve DNS by asking the root servers directly, instead of sending every lookup to one company. My README said this "bypasses ISP and Google DNS completely." During an audit I checked that claim and found it was false: Unbound was forwarding every lookup to a public resolver. I had configured recursion correctly, and a helper service was overriding it.

## How I found it

I asked Unbound what it was doing instead of trusting the configuration file:

```
sudo unbound-control list_forwards
```

A recursive resolver should list no forwarders. Mine listed a public resolver, plus the Pi-hole's own addresses.

## Root cause

- Debian ships a helper, `unbound-resolvconf.service`. It copies the system's own DNS server list into Unbound as forwarders.
- That list contained the public fallback resolver and the Pi's own addresses.
- Result: Unbound forwarded lookups instead of recursing. Listing the Pi itself as a forwarder also risked a lookup loop.
- My configuration file was correct the whole time. The override happened at runtime, which is why reading the file never revealed it.

## Fix

```
sudo systemctl disable --now unbound-resolvconf.service
sudo unbound-control forward off
```

## Verification

- `unbound-control list_forwards` shows no forwarders.
- A lookup through Unbound returns an answer with the `ad` (authenticated data) flag, which shows DNSSEC validation is active.
- A test domain with deliberately broken DNSSEC is refused.

## What I learned

- **Verify the running state, not only the configuration.** A file shows what I intended. The service's own status shows what is happening.
- **A working system can still be the wrong system.** Nothing failed and no alert fired. Only a deliberate check exposed the gap.
- **Defaults and helper services change behavior.** Packages add integrations I did not ask for, so I now review what gets enabled after an install.
- **Documentation is a claim that needs testing.** I corrected this README and now audit it against the real systems.

## Follow-ups

- [ ] Confirm the helper service stays disabled after the next reboot
- [ ] Add this check to a recurring audit checklist

## Security+ connection

Confidentiality and privacy, configuration management, configuration drift, and security assessment and auditing.
