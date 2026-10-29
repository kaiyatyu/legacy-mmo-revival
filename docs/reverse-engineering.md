# Reverse-engineering the client

Two problems: what the game *is* (its data) and how the client *talks* (its protocol). We solved the data first, because once you know every item and map, the protocol becomes much easier to read.

## Part 1: the data

### Find and open the archives

The install directory held a handful of large archive files plus a few loose config files. Steps:

1. **Inventory everything.** List every file with its size and its first bytes (the magic number). Group files by magic number; each group is one format.
2. **Crack the container.** Most game archives are a header, a file table (name, offset, size, maybe a compression flag) and the packed data. Look for the table by searching for file names you can guess (`.bmp`, `.wav`, map names). Some loose config files were encrypted with a fixed symmetric key compiled into the client. That key can be recovered from the executable, and the files then decrypt to plain text.
3. **Write one decoder per format** and dump everything to JSON. Keep the raw bytes of every record next to the decoded fields.

### Work out what each byte means

Fixed-size records (items, skills, monsters) are the bulk of the data. To learn a record's layout:

- **Anchor on what you can see.** Open the client, hover over an item and read its tooltip: name, level, price, attack. Find those values in the record. Each one pins down a field.
- **Diff similar records.** Two items that differ only in level differ in exactly the byte that holds level. Sort records by a suspected field and check that the order makes sense.
- **Follow the client code when stuck.** A tooltip renderer reads each field from a fixed offset. A disassembler on the routine that draws "Attack type" tells you exactly where that value lives, without guessing.
- **Record every finding in one place.** Each newly understood byte became one line in the raw reader, with a comment on how it was proven.

### Use the community's memory

Long-running games leave fan guides, wikis and forum posts behind. We collected the surviving ones (quest walkthroughs, skill tables, drop lists, event rules) as a second source. Where the client data and an old guide agree, we're confident. Where they disagree, the client wins for numbers and the guides win for behaviour the client can't show (how long jail lasted, how an event was scored).

## Part 2: the protocol

### Log everything, raw

The first server was a stub that accepted the connection and logged every byte, both directions, with a timestamp. Everything else followed from those logs.

1. **Find the framing.** Look for a length prefix: the first few bytes of each message predict where the next one starts.
2. **Find the encryption.** If the bytes look random after the length, the payload is encrypted or obfuscated. Compare messages that must be identical in plaintext (the same button pressed twice). Ours used a simple keyed scheme whose key setup could be read from the client.
3. **Separate the message type from the body.** Messages carry a type code (in our case a channel and a sub-opcode), then a body.

### Map opcodes by provoking them

Do one thing in the client (walk, open the bag, talk to an NPC, attack) and see which message it sends. Then answer it:

- **Silence first.** Send nothing back and watch the client: does it freeze, retry or carry on? That tells you whether it is waiting for a reply.
- **Echo, then vary.** Echo the message back, then change one field at a time and watch what the client draws. A health bar that moves when you change bytes 8–11 tells you those bytes are HP.
- **Keep a "no reply" log line** for every message the server doesn't handle yet. Sorted by count, that log is the to-do list for the protocol.

### Spawning the world

The hardest part was making the world appear: NPCs, monsters and other players. The client accepts several "spawn" messages, and only one of them makes an NPC clickable. We found it by spawning the same NPC with each candidate message and clicking it. Expect several of these "same data, different message" puzzles. The only way through is trying each candidate in the client.

### Characterise before you change

Before rewriting any working piece, capture what it currently does (packets in, packets out) as a test. Several times a "cleaner" rewrite silently dropped behaviour that only the old messy version had. Golden tests caught those regressions; reading the code never did.

## What we didn't need

- **The original server code.** Every rule we needed was either in the client data, observable in the protocol, or remembered in a guide.
- **Packet captures from the original servers.** They would have helped, but the client alone was enough. It is the server's most demanding, most specific consumer.
