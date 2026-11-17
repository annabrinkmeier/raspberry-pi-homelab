# Incident: DNS outage caused by a wrong system clock

**Date found:** November 15, 2026
**System:** Pi 2 (Pi-hole + Unbound + WireGuard)
**Impact:** Every DNS lookup sent to the Pi-hole failed
**Status:** Fixed, and verified across two reboots on November 17, 2026.

## Summary

After a reboot, my Pi-hole answered every DNS lookup with an error. The cause was the system clock, which was 55 days behind. The wrong clock broke DNSSEC validation, and the broken DNS then prevented the clock from correcting itself. Two services were each waiting on the other.

## What I saw

- Lookups through the Pi-hole returned `SERVFAIL` for every domain.
- Pi-hole and Unbound were both running, so a simple "is the service up" check looked healthy.
- `date` on the Pi showed a date 55 days in the past.

## Root cause

1. **A Raspberry Pi has no battery-backed clock.** When it boots, it starts from the last time it saved and relies on the network to correct it.
2. **The time service (chrony) was configured with server names**, such as a pool hostname. To use a name, it first needs a DNS lookup.
3. **DNS on this Pi goes through Unbound with DNSSEC validation.** DNSSEC signatures are only valid for a window of dates. With the clock 55 days behind, every signature looked invalid, so Unbound refused every answer.
4. **The loop:** chrony could not fix the clock without DNS, and DNS could not work without a correct clock.

```
wrong clock → DNSSEC validation fails → DNS returns SERVFAIL
     ↑                                          ↓
     └──────── time sync needs DNS ←────────────┘
```

## Fix

1. Set the clock by hand to the correct time.
2. Added time servers **by IP address** in `/etc/chrony/sources.d/`, so time sync no longer depends on DNS.
3. Restarted chrony and Unbound.
4. Verified: lookups resolve, a domain with deliberately broken DNSSEC is refused, and chrony reports it is synchronized.

A backup of the original chrony configuration was saved before the change.

## What I learned

- **Dependencies can form a circle.** Time needs DNS, and validated DNS needs time. Each one works alone; together they can deadlock after a reboot.
- **"The service is running" does not mean "the service is working."** Both daemons were active the whole time. The test that mattered was an actual lookup.
- **Check the clock early.** A wrong date breaks certificates, signatures, and log timelines. It now sits near the top of my troubleshooting list.
- **Security controls have failure modes.** DNSSEC did its job by refusing answers it could not verify. The lesson is to design for that, not to turn validation off.

## Follow-ups

- [x] Planned reboot to confirm the fix holds across a restart (done November 17: two reboots, clock synchronized within seconds each time)
- [ ] Add a real-time clock battery to the Pi 5 so the clock survives power loss
- [ ] Add a monitoring alert for clock drift

## Security+ connection

Availability (the "A" in the CIA triad), root cause analysis, DNSSEC, NTP, and resilience planning for single points of failure.
