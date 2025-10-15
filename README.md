# encrypt_user.js
// encrypt_users.js
// Node 16+ (no extra packages required)
const crypto = require('crypto');
const fs = require('fs');
const path = require('path');

// ---------- EDIT: plaintextUsers is set to your real JSON ----------
const plaintextUsers = [
  {
    "id": "6dyJ549OGt10cej72Be",
    "username": "admin_2Gt3jrz",
    "password": "2Gt3jrz",
    "expiresAt": "9999-99-99",
    "allowOffline": true
  },
  {
    "id": "b0cv09k1a",
    "username": "jrzmodz_gjhk123z",
    "password": "jrzmodz_gjhk123z",
    "expiresAt": "2025-11-4",
    "allowOffline": false
  }
];
// ---------- end edit --------------------------------------------------

// Build passphrase exactly as your client builds it
const passPieceA = "3k_";
const passPieceB = "y!-";
const passPieceC = "S3c";
const passPieceD = "r3T";
function buildPassphrase() {
  const combined = passPieceA + passPieceB + passPieceC + passPieceD;
  return combined.split("").reverse().join("");
}
const passphrase = buildPassphrase(); // matches client

// Derive key: SHA-256(passphrase) -> 32 bytes
function deriveKeyFromPassphrase(pass) {
  return crypto.createHash('sha256').update(pass, 'utf8').digest(); // 32 bytes
}

function encryptAesGcm(plainText, keyBytes) {
  const iv = crypto.randomBytes(12); // 12-byte IV
  const cipher = crypto.createCipheriv('aes-256-gcm', keyBytes, iv, { authTagLength: 16 });
  const ct1 = cipher.update(Buffer.from(plainText, 'utf8'));
  const ct2 = cipher.final();
  const tag = cipher.getAuthTag();
  const ciphertext = Buffer.concat([ct1, ct2]);
  // append tag to ciphertext for transport (common pattern)
  const ctAndTag = Buffer.concat([ciphertext, tag]);
  return {
    ivBase64: iv.toString('base64'),
    ctBase64: ctAndTag.toString('base64')
  };
}

function main() {
  const keyBytes = deriveKeyFromPassphrase(passphrase);
  const plain = JSON.stringify(plaintextUsers);
  const enc = encryptAesGcm(plain, keyBytes);
  const out = { ct: enc.ctBase64, iv: enc.ivBase64 };
  const outfile = path.join(process.cwd(), 'users.enc.json');
  fs.writeFileSync(outfile, JSON.stringify(out, null, 2), 'utf8');
  console.log('Wrote encrypted users to', outfile);
  console.log('Passphrase used (for reference):', passphrase);
  console.log('File contents:\n', JSON.stringify(out, null, 2));
}

main();
