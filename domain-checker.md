---
title: Domain Checker
tags: projects, technology
---

Plugin for the `taskrunner` suite. Cycles through the domains, running them against authoritative and global DNS servers to check for DNS propagation and SSL certification.

### Optimizations

#### First iteration:

- Check authoritative server.
- Check full global DNS pool.
- Update the `status` enum.
- Update the `lastResolvedStatus` timestamp.

#### If not resolved:

- Just do a full check.

#### If before threshold:

- ...and if the last status is `Resolved`, only check the authoritative server, player. If it's still resolved, cya! Otherwise, u got problems homie.

#### If past threshold of resolution:

- Check the authoritative, pimp, but only use a handful of globals.

#### Outstanding Tasks

- [x] Lower concurrency on DNS tasks to 3? Arbitrary
- [x] Improve logging on the taskrunner.
- [x] Add useful to console.time on lookups.
- [ ] Add a last resolved column in the table?
- [x]
