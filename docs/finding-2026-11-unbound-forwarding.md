# Audit finding: my "private" DNS resolver was forwarding to a public resolver

**Date found:** November 15, 2026
**System:** Pi 2 (Pi-hole + Unbound)
**Impact:** Privacy. DNS kept working, so nothing looked wrong.
**Status:** Fixed and verified across a reboot (November 17, 2026). The first fix was incomplete; see "Follow-up: the fix did not survive a reboot."

## Summary

I set up Unbound so my network would resolve DNS by asking the root servers directly, instead of sending every lookup to one company. My README said this "bypasses ISP and Google DNS completely." During an audit I checked that claim and found it was false: Unbound was forwarding every lookup to a public resolver. I had configured recursion correctly, and two separate mechanisms were overriding it at runtime. My first fix stopped only one of them. A planned reboot test two days later exposed the second.

## How I found it

I asked Unbound what it was doing instead of trusting the configuration file:

```
sudo unbound-control list_forwards
```

A recursive resolver should list no forwarders. Mine listed a public resolver, plus the Pi-hole's own addresses.

## Root cause (first mechanism)

- Debian ships a helper, `unbound-resolvconf.service`. It copies the system's own DNS server list into Unbound as forwarders.
- That list contained the public fallback resolver and the Pi's own addresses.
- Result: Unbound forwarded lookups instead of recursing. Listing the Pi itself as a forwarder also risked a lookup loop.
- My configuration file was correct the whole time. The override happened at runtime, which is why reading the file never revealed it.

## First fix (November 15)

```
sudo systemctl disable --now unbound-resolvconf.service
sudo unbound-control forward off
```

The forwarders disappeared and DNSSEC validation worked, so I marked the finding as fixed, with a reboot test as the one open follow-up.

## Follow-up: the fix did not survive a reboot (November 17)

After installing system updates I rebooted the Pi and ran my verification checks again. The forwarders were back, even though the helper service was still disabled.

**Second mechanism:** the `resolvconf` package runs every script in `/etc/resolvconf/update.d/` whenever the system's DNS server list changes. One of those scripts, `/etc/resolvconf/update.d/unbound`, does the same job as the service I had disabled: it pushes the system's DNS servers into Unbound as forwarders. It runs at boot, and again when WireGuard starts, because the WireGuard server config also registers a DNS server with `resolvconf`.

The boot log showed the hook firing:

```
journalctl -b | grep resolvconf
```

**Fix:** the script has a documented off switch, which survives package updates. I created `/etc/default/unbound` with:

```
USE_RESOLVCONF_FORWARDS=false
```

then cleared the forwarders again:

```
sudo unbound-control forward off
```

## Verification

- `unbound-control list_forwards` shows no forwarders.
- Running `sudo resolvconf -u`, which triggers the same hook as boot and WireGuard, leaves the forwarder list empty.
- **After a full reboot**, with both the boot trigger and the WireGuard trigger firing, the forwarder list is still empty.
- A lookup through Unbound returns an answer with the `ad` (authenticated data) flag, which shows DNSSEC validation is active. While Unbound was forwarding, that flag was missing.
- A test domain with deliberately broken DNSSEC is refused.

## What I learned

- **Verify the running state, not only the configuration.** A file shows what I intended. The service's own status shows what is happening.
- **A working system can still be the wrong system.** Nothing failed and no alert fired. Only a deliberate check exposed the gap.
- **One symptom can have more than one cause.** Stopping the first cause made the symptom disappear, which looked like a complete fix. It wasn't.
- **A fix isn't verified until it survives a restart.** Runtime changes such as `unbound-control forward off` are lost on reboot, and boot-time scripts can quietly undo them. The reboot test I had planned is what caught this.
- **Defaults and helper services change behavior.** Packages add integrations I did not ask for, so I now review what gets enabled after an install.
- **Documentation is a claim that needs testing.** I corrected the README, then corrected this write-up when the first fix turned out to be incomplete.

## Follow-ups

- [x] Confirm the forwarders stay off after a reboot (done November 17, after the second fix)
- [ ] Remove the unneeded `DNS =` line from the WireGuard server config, so WireGuard no longer touches the system DNS list
- [ ] Add `unbound-control list_forwards` to a recurring audit checklist, and alert if it is ever non-empty

## Security+ connection

Confidentiality and privacy, configuration management, configuration drift, root cause analysis, change verification, and security assessment and auditing.
