# Convert addresses

Within this document, we'll outline some examples on how to convert addresses between different formats in Titan Chain.

### Convert Hex <> Bech32 address

Titan Chain addresses are compatible with both Ethereum addresses (hex format) and Cosmos addresses (bech32 format). You can convert between the two formats easily using the same private key.

## Key Classes Overview

### PrivKeySecp256k1 Class

The `PrivKeySecp256k1` class handles private key operations:

```typescript
export class PrivKeySecp256k1 {
  // Generate a random private key
  static generateRandomKey(): PrivKeySecp256k1;

  // Create from Uint8Array
  constructor(protected readonly privKey: Uint8Array);

  // Convert to bytes
  toBytes(): Uint8Array;

  // Get corresponding public key
  getPubKey(): PubKeySecp256k1;

  // Sign a 32-byte digest
  signDigest32(digest: Uint8Array): {
    readonly r: Uint8Array;
    readonly s: Uint8Array;
    readonly v: number | null;
  };
}
```

### PubKeySecp256k1 Class

The `PubKeySecp256k1` class handles public key operations and address generation:

```typescript
export class PubKeySecp256k1 {
  // Create from compressed (33 bytes) or uncompressed (65 bytes) public key
  constructor(protected readonly pubKey: Uint8Array);

  // Get public key bytes (compressed or uncompressed)
  toBytes(uncompressed?: boolean): Uint8Array;

  // Generate Cosmos-style address (sha256 + ripemd160)
  getCosmosAddress(): Uint8Array;

  // Generate Ethereum-style address (keccak256)
  getEthAddress(): Uint8Array;

  // Verify a 32-byte digest signature
  verifyDigest32(digest: Uint8Array, signature: Uint8Array): boolean;
}
```

### Using TypeScript

You can easily convert between a Titan address and Ethereum address by using our utility functions:

```typescript
import { PrivKeySecp256k1 } from "lib/utils/key";
import { toBech32 } from "@cosmjs/encoding";
import { getAddress } from "ethers";

// From private key hex
const privateKeyHex = "your_private_key_hex";
const privateKey = new PrivKeySecp256k1(Buffer.from(privateKeyHex, "hex"));
const publicKey = privateKey.getPubKey();

// Generate ETH address (hex format)
const ethAddressBytes = publicKey.getEthAddress();
const ethAddress = getAddress(
  "0x" + Buffer.from(ethAddressBytes).toString("hex")
);

// Generate Cosmos address and convert to bech32
const cosmosAddressBytes = publicKey.getCosmosAddress();
const titanAddress = toBech32("titan", cosmosAddressBytes);

console.log("ETH address => ", ethAddress);
console.log("Titan address => ", titanAddress);
```

### **Convert ETH Address to Titan Bech32**

For chains using ETH-style addressing with bech32 encoding (slip44 = 60):

```typescript
import { PrivKeySecp256k1 } from "lib/utils/key";
import { toBech32 } from "@cosmjs/encoding";

const privateKey = new PrivKeySecp256k1(Buffer.from(privateKeyHex, "hex"));
const publicKey = privateKey.getPubKey();

// ETH address bytes to bech32 format
const ethAddressBytes = publicKey.getEthAddress();
const ethToBech32 = toBech32("titan", ethAddressBytes);

console.log("ETH address in bech32 format => ", ethToBech32);
```

### **Generate Random Key and Addresses**

Create a new random private key and generate all address formats:

```typescript
import { PrivKeySecp256k1 } from "lib/utils/key";
import { toBech32 } from "@cosmjs/encoding";
import { getAddress } from "ethers";

// Generate random private key
const privateKey = PrivKeySecp256k1.generateRandomKey();
const publicKey = privateKey.getPubKey();

// Get private key as hex
const privateKeyHex = Buffer.from(privateKey.toBytes()).toString("hex");

// Generate all address formats
const ethAddressBytes = publicKey.getEthAddress();
const ethAddress = getAddress(
  "0x" + Buffer.from(ethAddressBytes).toString("hex")
);

const cosmosAddressBytes = publicKey.getCosmosAddress();
const titanAddress = toBech32("titan", cosmosAddressBytes);

console.log({
  privateKeyHex,
  ethAddress,
  titanAddress,
});
```

### **Multi-chain Address Generation**

Generate addresses for multiple chains using the same private key:

```typescript
import { PrivKeySecp256k1 } from "lib/utils/key";
import { toBech32 } from "@cosmjs/encoding";
import { getAddress } from "ethers";

function generateAddresses(privateKeyHex: string) {
  const privateKey = new PrivKeySecp256k1(Buffer.from(privateKeyHex, "hex"));
  const publicKey = privateKey.getPubKey();

  const addressMap = new Map<string, string>();

  // ETH address for EVM chains
  const ethAddressBytes = publicKey.getEthAddress();
  const ethAddress = getAddress(
    "0x" + Buffer.from(ethAddressBytes).toString("hex")
  );

  // Cosmos address for Cosmos chains
  const cosmosAddressBytes = publicKey.getCosmosAddress();

  // Chain-specific addresses
  const chains = [
    { chainId: "eip155:1", prefix: null, type: "eip155" },
    { chainId: "titan_18888-1", prefix: "titan", type: "cosmos", slip44: 60 },
    { chainId: "cosmoshub-4", prefix: "cosmos", type: "cosmos", slip44: 118 },
  ];

  chains.forEach((chain) => {
    if (chain.type === "eip155") {
      addressMap.set(chain.chainId, ethAddress);
    } else if (chain.type === "cosmos") {
      const address =
        chain.slip44 === 60
          ? toBech32(chain.prefix, ethAddressBytes)
          : toBech32(chain.prefix, cosmosAddressBytes);
      addressMap.set(chain.chainId, address);
    }
  });

  return addressMap;
}

const addresses = generateAddresses("your_private_key_hex");
console.log("Multi-chain addresses => ", Object.fromEntries(addresses));
```

### **Signing and Verification**

Sign and verify messages using the secp256k1 curve:

```typescript
import { PrivKeySecp256k1 } from "lib/utils/key";
import { sha256 } from "@noble/hashes/sha2";

const privateKey = PrivKeySecp256k1.generateRandomKey();
const publicKey = privateKey.getPubKey();

// Message to sign
const message = "Hello Titan Chain";
const messageBytes = new TextEncoder().encode(message);
const digest = sha256(messageBytes);

// Sign the digest
const signature = privateKey.signDigest32(digest);

// Verify the signature
const signatureBytes = new Uint8Array([...signature.r, ...signature.s]);
const isValid = publicKey.verifyDigest32(digest, signatureBytes);

console.log("Signature valid:", isValid);
```

### **Key Differences**

* **ETH addressing**: Uses uncompressed public key + keccak256 hash
* **Cosmos addressing**: Uses compressed public key + sha256 + ripemd160 hash
* **Chain-specific logic**: Some chains use ETH addressing even with bech32 format (slip44 = 60)
* **Key generation**: Uses secp256k1 curve for both private and public keys
* **Signing**: Implements deterministic ECDSA with lowS for compatibility

{% hint style="info" %}
The same private key generates different address formats, allowing users to interact with multiple chains in the Titan ecosystem.
{% endhint %}
