---
name: e2e-encrypted-local-first
description: Use for E2E zero-knowledge sync / client-side WebCrypto.
---

# E2E-encrypted (zero-knowledge) local-first sync

## When to Use
- User wants cross-device sync but worries the server/app-owner could read the data ("จะเก็บข้อมูลข้ามเครื่องได้แต่ไม่มาเก็บที่เรา / ให้คนใช้รู้สึกปลอดภัยในข้อมูล").
- Building client-side encryption (WebCrypto) or zero-knowledge sync.
- Hit the TS build error `Type 'ArrayBufferLike' is not assignable to type 'ArrayBuffer' ... SharedArrayBuffer` with `crypto.subtle`.

Goal: sync data across a user's devices, but the server/app-owner can never read it. Store ciphertext on a dumb object store (Supabase Storage / R2 / S3 / D1); only the user's passphrase can decrypt.

## Pattern
1. Data lives **on device** (local-first). Encrypt it there before any upload.
2. Key = derived from a **user passphrase** (never sent to server): PBKDF2 → AES-GCM-256.
3. Upload only an `Envelope { salt, iv, data(base64) }` — server sees ciphertext only.
4. Second device downloads the envelope and decrypts with the **same passphrase**.

Use the browser-native **Web Crypto** (`crypto.subtle`). No dependency needed.

```ts
// core (browser-safe; also runs on Node 22 via global crypto.subtle)
function toB64(b: Uint8Array): string { let s=''; for (let i=0;i<b.length;i++) s+=String.fromCharCode(b[i]); return btoa(s); }
function fromB64(b64: string): Uint8Array<ArrayBuffer> {   // <-- typed on purpose
  const s = atob(b64); const out = new Uint8Array(s.length);
  for (let i=0;i<s.length;i++) out[i]=s.charCodeAt(i); return out;
}
async function deriveKey(pass: string, salt: Uint8Array<ArrayBuffer>) {
  const base = await crypto.subtle.importKey('raw', new TextEncoder().encode(pass), 'PBKDF2', false, ['deriveKey']);
  return crypto.subtle.deriveKey({ name:'PBKDF2', salt, iterations:100_000, hash:'SHA-256' }, base,
    { name:'AES-GCM', length:256 }, false, ['encrypt','decrypt']);
}
export async function encryptText(pass: string, plain: string) {
  const salt=crypto.getRandomValues(new Uint8Array(16)), iv=crypto.getRandomValues(new Uint8Array(12));
  const key=await deriveKey(pass,salt);
  const c=await crypto.subtle.encrypt({name:'AES-GCM',iv}, key, new TextEncoder().encode(plain));
  return { salt:toB64(salt), iv:toB64(iv), data:toB64(new Uint8Array(c)) };
}
export async function decryptText(pass: string, e:{salt:string;iv:string;data:string}) {
  const key=await deriveKey(pass, fromB64(e.salt));
  const p=await crypto.subtle.decrypt({name:'AES-GCM', iv:fromB64(e.iv)}, key, fromB64(e.data));
  return new TextDecoder().decode(p);
}
```

## The TypeScript pitfall (real, bites every time)
- `crypto.subtle.*` `BufferSource` params require **`ArrayBufferView<ArrayBuffer>`**.
- `Uint8Array.from(...)` is typed **`Uint8Array<ArrayBufferLike>`** → "Type 'ArrayBufferLike' is not assignable to type 'ArrayBuffer' ... SharedArrayBuffer" at build.
- Fix: allocate with `new Uint8Array(n)` + index assign (returns `Uint8Array<ArrayBuffer>`), and type helper/param signatures as `Uint8Array<ArrayBuffer>`.
- `crypto.getRandomValues(new Uint8Array(n))` already returns the right type.

## Node 22 gotcha
`globalThis.crypto` is **getter-only** — `globalThis.crypto = webcrypto` throws. Don't override: Node 22 already exposes `crypto.subtle`. Tests run fine via `tsx` without any setup.

## Tests (keep one)
Assert: roundtrip (same pass → same plaintext); **envelope contains no plaintext substring**; **wrong passphrase throws**.
Run test files with `tsx` (not tsc) and **exclude `*.test.ts` from the tsc build** (they reference node globals/`process` that aren't in the DOM lib) — `"exclude": ["src/**/*.test.ts"]`.

## Hard truth to surface to the user
**Lost passphrase = permanent data loss** (you cannot recover it — that's the point of zero-knowledge). Always add a clear UX warning + a passphrase-backup path.

## When NOT to use
- If the app legitimately needs server-side search/aggregation over the data → E2E blocks that. Use it only when "even we can't read it" is the selling point.
- Not for health/regulated data where server-side audit/reporting is required.
