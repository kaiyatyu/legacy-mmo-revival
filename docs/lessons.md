# Lessons

What we'd tell anyone reviving a game from its client.

**Integration bugs beat unit bugs.** Almost every bug players hit was in the seams: a reward granted in the database but never sent to the client, a packet sent in the wrong order, a feature that was built but unreachable because nothing ever called it. Test with a fake client end to end: packets in, packets out.

**Check what the client can actually hold.** A premium membership that stacked to 60 days crashed a reply, because the client's timer field is an unsigned 32-bit count of milliseconds and tops out at about 49.7 days. Every field you send has a width the client decided long ago. Clamp at the edge.

**Never rewrite working code without golden tests.** A "clean" rebuild of a messy module quietly dropped behaviour players relied on. Characterise first, then change.

**Debug logging has a cost.** Logging every raw packet was essential while we decoded the protocol. Months later it was 80% of an 880 MB-a-day log. Put protocol dumps behind a switch, and make log rotation size-based, not only daily.

**Tests that share state run one at a time.** Our tests share one database copy. Two runs at once produced fake failures and a corrupted file. A test suite you can't trust is worse than none.

**Keep the original design as the default.** Players ask for reasonable changes every day. Each is small; together they turn the game into something nobody designed. Route every request through the design map, deviate only to save players time, and log every deviation.

**Hand the community real decisions.** Simple yes/no polls on rule changes gave players ownership and removed accusations of favouritism. Two rules mattered most. The owner never gives or lends gear, even for testing. Any PvP change is voted on by every job, not just the one that benefits.

**Keep private things private.** Never post server addresses, account names or anyone's equipment publicly. Publish only what helps players: the upgrade success table, yes; internal damage formulas, no.
