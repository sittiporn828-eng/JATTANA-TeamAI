# E2E-encrypted sync — working crypto lib (zero-knowledge)

WebCrypto (no deps): AES-GCM 256 + PBKDF2 key-from-passphrase. Server only ever stores the `Envelope` (ciphertext); it cannot decrypt without the user's passphrase. Verified with a roundtrip test that also asserts the envelope leaks no plaintext and a wrong passphrase fails.

## `src/lib/crypto.ts`

```ts
// key มาจาก passphrase ผู้ใช้, ไม่เคยส่ง key ขึ้น server (zero-knowledge)
const enc = new TextEncoder()
const dec = new TextDecoder()

function toB64(bytes: Uint8Array): string {
  let s = ''
  for (let i = 0; i < bytes.length; i++) s += String.fromCharCode(bytes[i]) // อย่า spread — ใต้ ES5/downlevelIteration พัง
  return btoa(s)
}
function fromB64(b64: string): Uint8Array<ArrayBuffer> { // ต้อง typed Uint8Array<ArrayBuffer> ให้ WebCrypto รับ
  const s = atob(b64)
  const out = new Uint8Array(s.length)
  for (let i = 0; i < s.length; i++) out[i] = s.charCodeAt(i)
  return out
}

async function deriveKey(passphrase: string, salt: Uint8Array<ArrayBuffer>): Promise<CryptoKey> {
  const base = await crypto.subtle.importKey('raw', enc.encode(passphrase), 'PBKDF2', false, ['deriveKey'])
  return crypto.subtle.deriveKey(
    { name: 'PBKDF2', salt, iterations: 100_000, hash: 'SHA-256' },
    base,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt', 'decrypt'],
  )
}

export interface Envelope { salt: string; iv: string; data: string } // ทั้งหมด base64

export async function encryptText(passphrase: string, plaintext: string): Promise<Envelope> {
  const salt = crypto.getRandomValues(new Uint8Array(16))
  const iv = crypto.getRandomValues(new Uint8Array(12))
  const key = await deriveKey(passphrase, salt)
  const cipher = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, key, enc.encode(plaintext))
  return { salt: toB64(salt), iv: toB64(iv), data: toB64(new Uint8Array(cipher)) }
}

export async function decryptText(passphrase: string, e: Envelope): Promise<string> {
  const key = await deriveKey(passphrase, fromB64(e.salt))
  const plain = await crypto.subtle.decrypt({ name: 'AES-GCM', iv: fromB64(e.iv) }, key, fromB64(e.data))
  return dec.decode(plain)
}
```

## Test (`src/lib/crypto.test.ts`) — run with `tsx`

Node 22 has `globalThis.crypto` (getter-only) already; do NOT reassign it. Assert roundtrip, envelope contains no plaintext, wrong passphrase rejects.

```ts
import { encryptText, decryptText } from './crypto'
async function assert(c: unknown, m: string) { if (!c) throw new Error('FAIL: ' + m) }

async function main() {
  const secret = 'ข้อมูลส่วนตัว' + Math.random()
  const env = await encryptText('my-passphrase', secret)
  await assert((await decryptText('my-passphrase', env)) === secret, 'roundtrip')
  await assert(!env.data.includes(secret), 'no plaintext in envelope')
  let failed = false
  try { await decryptText('wrong-pass', env) } catch { failed = true }
  await assert(failed, 'wrong passphrase must fail')
  console.log('crypto E2E tests OK')
}
main().catch((e) => { console.error(e); process.exit(1) })
```

## tsconfig (build must not choke on tests)

```json
{ "include": ["src"], "exclude": ["src/**/*.test.ts"] }
```
Tests use node globals (`process`) → excluded from `tsc -b` so no `@types/node` needed; run them with `npx tsx src/lib/crypto.test.ts`.

## Sync architecture

```
เครื่อง A: ข้อมูล → encrypt(passphrase) → upload Envelope ขึ้น Storage ☁️  (server เห็นแต่ ciphertext)
เครื่อง B: download Envelope → decrypt(passphrase เดียวกัน) → ข้อมูลครบ
```
Store the whole dataset as one JSON blob, encrypt it, sync the single Envelope. Lost passphrase = permanent data loss — surface a clear warning + passphrase backup flow.
