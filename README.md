# CyberHawk trust roots

One file per day. Each one fixes the fingerprint of every CyberHawk client
record as it stood that morning, at a time nobody can change afterwards.

This repository exists so that verifying a CyberHawk trust page does not
require trusting CyberHawk.

## What is in a file

```json
{
  "service": "CyberHawk",
  "localDate": "2026-09-14",
  "root": "a sha256 over every client chain head that day",
  "companies": 3,
  "postedAt": "2026-09-14T12:00:04.000Z",
  "signature": "base64 Ed25519 over signedBytes",
  "algorithm": "Ed25519",
  "signedBytes": "the exact text the signature covers"
}
```

No client names. No client data. A date, a hash, a count, a signature.

## How to check one yourself

**1. Check the signature.** The public key is below. It covers `signedBytes`
byte for byte.

```bash
# node
node -e '
const {webcrypto:c}=require("node:crypto"),f=require("fs");
(async()=>{
  const p=JSON.parse(f.readFileSync(process.argv[1],"utf8"));
  const k=await c.subtle.importKey("raw",
    Buffer.from("HiXIuR5a7Aeh3xYNpv/GRNMrJPf4HLDfZBAWNseJGWM=","base64"),{name:"Ed25519"},false,["verify"]);
  console.log(await c.subtle.verify("Ed25519",k,
    Buffer.from(p.signature,"base64"),
    new TextEncoder().encode(p.signedBytes)) ? "signature ok" : "SIGNATURE BAD");
})();' roots/2026-09-14.json
```

**2. Check the root is the one being served.** Fetch
`https://gocyberhawk.com/trust-root.json`, hash its inputs, and compare with
`root` here:

```
sha256( each "<hashed company id>:<chain head>" joined by newlines,
        sorted by the hashed id )
```

If CyberHawk ever rewrote a client's record, that sum would stop matching the
number in this repository, and the commit date proves the number was here
first.

The trust page's own **Verify** button does all of this in your browser.

## The public key

```
HiXIuR5a7Aeh3xYNpv/GRNMrJPf4HLDfZBAWNseJGWM=
```

Ed25519, raw, base64. If this key ever changes, both keys will be listed here
with the dates each one applies from.

## Why so little

Deliberately. What is published has to be enough to catch a rewrite and no more
than that. The inputs live on the service, where anyone can fetch them; the
number that pins them down lives here, where the service cannot reach.
