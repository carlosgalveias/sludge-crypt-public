# 🔐 SLUDGE - Secure Layer Using Dynamic Gunk Encryption

[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg)](https://opensource.org/licenses/ISC)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org/)
[![WebAssembly](https://img.shields.io/badge/WebAssembly-Ready-orange.svg)](https://webassembly.org/)

**SLUDGE-Layer Application Security (SLAS)** is a novel encryption system that combines ECDH key exchange, AES-256-GCM symmetric encryption, and WebAssembly-based gunk fermentation mechanics for time-dependent key derivation. The result is a multi-layered security approach that provides perfect forward secrecy with built-in obfuscation.

## 🌟 Key Features

- **🔑 ECDH P-256 Key Exchange** - Elliptic curve cryptography for secure key agreement
- **🔒 AES-256-GCM Encryption** - Industry-standard authenticated encryption
- **⚛️ Bio-Sludge Key Derivation** - Unique time-based key material using internal gunk mutation cycles
- **⚡ WebAssembly Acceleration** - Performance-optimized cryptographic calculations
- **🛡️ Perfect Forward Secrecy** - Each session uses ephemeral keys
- **🎯 Zero-Trust Architecture** - No shared secrets, no persistent keys
- **🔄 Simple API** - Just two functions: `compose()` and `decompose()`

---

## 📊 Why SLUDGE? Comparison with Standard Libraries

### Advantages Over Traditional Encryption

| Feature | SLUDGE | crypto-js | bcrypt | node:crypto |
|---------|--------|-----------|--------|-------------|
| **ECDH Key Exchange** | ✅ Built-in | ❌ Manual | ❌ N/A | ✅ Manual |
| **Time-Based Key Derivation** | ✅ Gunk Matrix | ❌ No | ❌ No | ❌ No |
| **WebAssembly Obfuscation** | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Perfect Forward Secrecy** | ✅ Automatic | ⚠️ Manual | ❌ N/A | ⚠️ Manual |
| **Authenticated Encryption** | ✅ AES-GCM | ⚠️ Various | ❌ Hashing only | ✅ Various |
| **Simple API** | ✅ 2 functions | ❌ Complex | ✅ Simple | ❌ Complex |

### 🎯 When to Use SLUDGE

**SLUDGE excels in scenarios requiring:**

- **Application-level encryption** where you need end-to-end security
- **Time-sensitive data** that benefits from temporal key derivation
- **Zero-knowledge architecture** where the server never sees plaintext
- **Obfuscated cryptography** to resist reverse engineering
- **Perfect forward secrecy** without complex key management

**Traditional libraries are better for:**

- Password hashing (use bcrypt/argon2)
- TLS/SSL connections (use native TLS)
- Simple symmetric encryption without key exchange

---

## 🚀 Quick Start

### Installation

```bash
npm install sludge-crypt
```

### Basic Usage

```javascript
const { compose, decompose, generateBilePair } = require('sludge-crypt');

// Generate key pairs for sender and recipient
const recipientKeys = await generateBilePair();

// Compose (encrypt) data
const secretData = {
  message: "Top secret information",
  level: "classified"
};

const encrypted = await compose(
  secretData,
  recipientKeys.publicKey,
  Date.now()
);

console.log('Encrypted:', encrypted);

// Decompose (decrypt) data
const decrypted = await decompose(
  encrypted,
  recipientKeys.privateKey
);

console.log('Decrypted:', decrypted);
// Output: { message: "Top secret information", level: "classified" }
```

---

## 🔄 Bidirectional Encryption (Client ↔ Server)

### Understanding Two-Way Secure Communication

A common question: **"If the client encrypts with the server's public key, how does the server send an encrypted response back?"**

**Answer**: Both parties need their own key pairs! ECDH enables true bidirectional encryption.

### The Pattern

```javascript
// SETUP: Both parties generate key pairs
const clientKeys = await generateBilePair();
const serverKeys = await generateBilePair();

// KEY EXCHANGE: Share public keys (safe over any channel)
// Client sends: clientKeys.publicKey to server
// Server sends: serverKeys.publicKey to client

// CLIENT → SERVER: Encrypt with server's public key
const request = { action: "getData", userId: 123 };
const encryptedRequest = await compose(request, serverKeys.publicKey);

// SERVER: Decrypt with its own private key
const decryptedRequest = await decompose(encryptedRequest, serverKeys.privateKey);

// SERVER → CLIENT: Encrypt with client's public key
const response = { status: "success",  {...} };
const encryptedResponse = await compose(response, clientKeys.publicKey);

// CLIENT: Decrypt with its own private key
const decryptedResponse = await decompose(encryptedResponse, clientKeys.privateKey);
```

### Key Principles

| Concept | Explanation |
|---------|-------------|
| **Each party has their own key pair** | Client has clientPrivate + clientPublic; Server has serverPrivate + serverPublic |
| **Encryption uses recipient's PUBLIC key** | To send to server: use serverPublicKey; To send to client: use clientPublicKey |
| **Decryption uses your own PRIVATE key** | Server decrypts with serverPrivateKey; Client decrypts with clientPrivateKey |
| **Public keys are safe to transmit** | Can be sent over HTTP, WebSocket, stored in databases |
| **Private keys NEVER leave their owner** | Each party keeps their private key secret |
| **ECDH derives same shared secret** | Both sides independently calculate the same encryption key |

---

## 📚 API Reference

### `compose(data, publicKey, timestamp)`

Encrypts data using the SLUDGE encryption system.

**Parameters:**
- `data` **(Object|string)** - The plaintext data to encrypt. Can be any JSON-serializable object or string.
- `publicKey` **(Object)** - The recipient's ECDH public key in JWK format.
- `timestamp` **(number, optional)** - Unix timestamp in milliseconds for Gunk Matrix calculation. Defaults to `Date.now()`.

**Returns:**
- `Promise<string>` - JSON string containing the encrypted package with structure:
  - `p`: Ephemeral public key (for ECDH key exchange)
  - `i`: Initialization vector (12 bytes)
  - `d`: Encrypted data (AES-256-GCM ciphertext)
  - `t`: Timestamp used for encryption

**Example:**

```javascript
const { compose, generateBilePair } = require('sludge-crypt');

const recipientKeys = await generateBilePair();

const data = {
  creditCard: '4532-1234-5678-9010',
  cvv: '123',
  expiry: '12/25'
};

const encrypted = await compose(data, recipientKeys.publicKey);
console.log(encrypted);
// Output: {"p":{...},"i":[...],"d":[...],"t":1708800000000}
```

**Throws:**
- `Error` - If data is null/undefined, publicKey is invalid, or encryption fails

---

### `decompose(encryptedData, privateKey, timestamp)`

Decrypts data that was encrypted with `compose()`.

**Parameters:**
- `encryptedData` **(string)** - The encrypted payload JSON string from `compose()`.
- `privateKey` **(Object)** - The recipient's ECDH private key in JWK format (must match the public key used for encryption).
- `timestamp` **(number, optional)** - Override timestamp for decryption. Normally not needed as the timestamp is embedded in the encrypted package.

**Returns:**
- `Promise<Object|string>` - The decrypted plaintext data in its original form.

**Example:**

```javascript
const { decompose } = require('sludge-crypt');

const encryptedPackage = '{"p":{...},"i":[...],"d":[...],"t":1708800000000}';

const decrypted = await decompose(encryptedPackage, recipientKeys.privateKey);
console.log(decrypted);
// Output: { creditCard: '4532-1234-5678-9010', cvv: '123', expiry: '12/25' }
```

**Throws:**
- `Error` - If encryptedData is invalid JSON, privateKey doesn't match, or decryption fails

---

### `generateBilePair()`

Generates a new ECDH key pair for use with `compose()` and `decompose()`.

**Parameters:**
- None

**Returns:**
- `Promise<Object>` - Object containing:
  - `publicKey` **(Object)** - ECDH P-256 public key in JWK format (share with senders)
  - `privateKey` **(Object)** - ECDH P-256 private key in JWK format (keep secret)

**Example:**

```javascript
const { generateBilePair } = require('sludge-crypt');

const keyPair = await generateBilePair();

console.log('Public Key (share this):', keyPair.publicKey);
// Output: { kty: 'EC', crv: 'P-256', x: '...', y: '...' }

console.log('Private Key (keep secret):', keyPair.privateKey);
// Output: { kty: 'EC', crv: 'P-256', x: '...', y: '...', d: '...' }
```

---

## 🏗️ Architecture

### Multi-Layered Security Approach

SLUDGE uses a defense-in-depth strategy with four distinct security layers:

```
┌─────────────────────────────────────────────────────────┐
│                    Layer 1: ECDH                        │
│  Elliptic Curve Diffie-Hellman Key Exchange (P-256)    │
│  ► Ephemeral key pairs for each encryption              │
│  ► Perfect forward secrecy                              │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│            Layer 2: Gunk Matrix                     │
│  Time-Based Key Derivation Using Gunk Fermentation Paths   │
│  ► Simulates 9 mutant fermentation trajectory chains         │
│  ► Calculates 9x9x3 gunk entropy matrix             │
│  ► SHA-256 hash creates unique salt                     │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│               Layer 3: HKDF                             │
│  HMAC-based Key Derivation Function                     │
│  ► Combines ECDH shared secret + Gunk Matrix salt   │
│  ► Derives AES-256 encryption key                       │
│  ► Info string: vomit-trail                             │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│            Layer 4: AES-256-GCM                         │
│  Advanced Encryption Standard (Galois/Counter Mode)     │
│  ► 256-bit key length                                   │
│  ► Authenticated encryption (prevents tampering)        │
│  ► Random 12-byte IV per encryption                     │
└─────────────────────────────────────────────────────────┘
```

### The Gunk Matrix Explained

> [!IMPORTANT]
> **TECHNICAL CLARIFICATION:** The following technical description of "Gunk Decay" utilizes proprietary terminology to represent the underlying algorithmic architecture. These conceptual frameworks serve as an abstracted mapping for the mathematical models employed and may not reflect literal physical or implementation-level characteristics. 

The Gunk Matrix is a unique time-based key derivation mechanism inspired by bio-sludge mutant trajectories through a simulated sewer ecosystem. It simulates the fermentation paths of 9 different mutant chains, where each chain has specific reaction parameters (viscosity, growth constant, mutation ratio, etc.). Gunk states are calculated based on the current timestamp and create a 9×9×3 matrix of entropy values (243 values), which is then hashed with SHA-256 to produce a deterministic salt.

**Why This Works:**
1. **Deterministic** - Same timestamp always produces the same matrix
2. **Time-dependent** - Different timestamps produce different matrices  
3. **Complex** - 243 floating-point calculations create high entropy
4. **Obfuscated** - Difficult to reverse-engineer the key derivation
5. **Synchronized** - Both parties can independently calculate the same matrix

**Time Window:** Gunk Matrix recalculates every 2 minutes (120,000 ms), allowing for clock drift between systems and providing time-limited decryption capability.

---

## 📊 Performance Benchmark

**CPU:** Intel(R) Core(TM) i7-4700HQ CPU @ 2.40GHz

| Payload Size | Compose (Avg) | Decompose (Avg) | Round-trip (Avg) |
|--------------|---------------|-----------------|------------------|
| 100 KB       | 32.12 ms      | 25.23 ms        | 57.34 ms         |
| 256 KB       | 77.40 ms      | 67.21 ms        | 144.61 ms        |
| 512 KB       | 161.01 ms     | 122.65 ms       | 283.66 ms        |
| 1024 KB      | 320.39 ms     | 244.56 ms       | 564.95 ms        |
| 2048 KB      | 664.74 ms     | 495.56 ms       | 1160.30 ms       |

*Measured over 25 iterations (5 warmup).*

---

## 🔐 Security Considerations

### Zero-Trust Model

- ✅ No pre-shared keys - all keys generated per-session
- ✅ Ephemeral ECDH keys - new pair for each encryption
- ✅ Perfect forward secrecy - compromising one session doesn't affect others
- ✅ Authenticated encryption (AES-GCM) prevents tampering
- ✅ Server never has ability to decrypt user data

### Best Practices

**DO:**
- Rotate keys regularly
- Validate timestamps are within acceptable range
- Store private keys securely
- Always use HTTPS for transmission
- Implement rate limiting
- Log decryption failures

**DON'T:**
- Don't reuse ephemeral keys
- Don't store private keys in plain text
- Don't trust client timestamps without validation
- Don't use over insecure channels
- Don't expose private keys in logs or errors

### Key Rotation Recommendations

- **Session Keys:** Generate new ephemeral keys for every encryption
- **User Keys:** Rotate every 90 days or on suspected compromise
- **API Keys:** Rotate quarterly
- **Emergency Rotation:** Immediate rotation if breach suspected

---

## 🛠️ Troubleshooting

### Common Issues

**Issue: `Module not found: sludge-crypt`**
```bash
# Solution: Ensure the package is installed
npm install sludge-crypt

# Then import it
const sludge = require('sludge-crypt');
```

**Issue: `Decryption failed` or `wrong key` errors**
- Verify you're using the correct private key that matches the public key used for encryption
- Check timestamp synchronization between systems (within 2-minute window)
- Ensure encrypted data wasn't corrupted during transmission

**Issue: Performance is slower than expected**
- Pre-generate and cache key pairs when possible
- Use batch operations for multiple encryptions
- Ensure WASM is loading correctly

**Issue: Browser compatibility**
- Requires modern browser with WebCrypto API support
- Ensure `window.crypto.subtle` is available
- Test in Chrome 60+, Firefox 57+, Safari 11+, Edge 79+

---

## 📄 License

ISC License

Copyright (c) 2026, Carlos Galveias

Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

---

### Code Style

- Use 4-space indentation
- Follow existing naming conventions
- Add JSDoc comments for new functions
- Write tests for new features

---

## 📞 Support

For issues, questions, or contributions:
- **GitHub:** https://github.com/carlosgalveias/sludge-crypt-public
- **Issues:** https://github.com/carlosgalveias/sludge-crypt-public/issues
- **Documentation:** Full API reference in this README

---

## 🏆 Acknowledgments

- Inspired by Signal Protocol's double ratchet algorithm
- Gunk fermentation calculations based on mutant trajectory chemistry
- Built with Emscripten WebAssembly toolchain

---

## ⚖️ Legal Disclaimer and Technical Usage

This software is provided "as is" and any express or implied warranties, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose are disclaimed. 

**Technical Accuracy & Proprietary Mapping:**
The descriptions provided in this documentation, including references to "Gunk Decay", "Matrix Trajectories", and associated scientific terminology, are utilized as a proprietary conceptual mapping of the library's internal logic. These terms are meant for organizational and architectural identification and may not correspond to literal mathematical or physical phenomena. The developer provides no guarantee regarding the technical accuracy of these conceptual descriptions beyond their functional performance within the Sludge-Crypt environment.

**Implementation Notice:**
Implementation details, including entropy derivation methods and algorithmic structures, are subject to modification without notice to maintain system integrity and optimization. It is the responsibility of the end-user to perform a comprehensive security audit of the implementation before production deployment.

**Limitation of Liability:**
In no event shall the developer or contributors be liable for any direct, indirect, incidental, special, exemplary, or consequential damages (including, but not limited to, procurement of substitute goods or services; loss of use, data, or profits; or business interruption) however caused and on any theory of liability, whether in contract, strict liability, or tort (including negligence or otherwise) arising in any way out of the use of this software, even if advised of the possibility of such damage.

For critical security applications, please consult with qualified cryptography experts and conduct thorough security audits.

---

**Made with 🔐 for application-level security**