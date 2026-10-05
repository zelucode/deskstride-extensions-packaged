# Data Encryption / Crypto Utilities Extension

**Version:** 1.1.1

A cryptographic utilities extension for DeskStride providing file/string encryption, digital signatures, key generation, and secure key derivation.

## Features

- **Encrypt/Decrypt File**: Symmetric file encryption with password-based key derivation
- **Encrypt/Decrypt String**: Symmetric string encryption for secure data handling
- **Generate Key Pair**: Asymmetric key generation (RSA, Ed25519) for signing/encryption
- **Sign Data**: Digital signatures for data authentication and integrity
- **Verify Signature**: Signature verification using public keys
- **Derive Key**: Secure key derivation from passwords (PBKDF2, Argon2)

## Setup

No external setup required. The extension includes all necessary cryptographic libraries via the `cryptography` package.

### Extension Settings

Configure in the Extensions page:

- **Default Encryption Algorithm**: Default algorithm for encryption operations (AES-GCM, ChaCha20-Poly1305, Fernet)
- **Default Key Derivation**: Default key derivation function (PBKDF2, Argon2)

## Nodes

### Encrypt File
Encrypt a file using symmetric encryption with password-based key derivation.

**Inputs:**
- File Path (required): Path to the file to encrypt
- Output Path: Path for encrypted file (default: original.encrypted)
- Password (required): Password for encryption
- Algorithm: Encryption algorithm (AES-GCM, ChaCha20-Poly1305, Fernet)

**Outputs:**
- success: Boolean indicating if encryption succeeded
- inputPath: Original file path
- outputPath: Encrypted file path
- algorithm: Algorithm used
- fileSize: Size of original file
- status: Status message

### Decrypt File
Decrypt a file using symmetric encryption with password-based key derivation.

**Inputs:**
- File Path (required): Path to the encrypted file
- Output Path: Path for decrypted file (default: removes .encrypted)
- Password (required): Password for decryption
- Algorithm: Encryption algorithm used

**Outputs:**
- success: Boolean indicating if decryption succeeded
- inputPath: Encrypted file path
- outputPath: Decrypted file path
- algorithm: Algorithm used
- fileSize: Size of decrypted file
- status: Status message

### Encrypt String
Encrypt a string using symmetric encryption with password-based key derivation.

**Inputs:**
- Plaintext (required): The string to encrypt
- Password (required): Password for encryption
- Algorithm: Encryption algorithm (AES-GCM, ChaCha20-Poly1305, Fernet)

**Outputs:**
- success: Boolean indicating if encryption succeeded
- ciphertext: Encrypted string (base64 encoded)
- algorithm: Algorithm used
- status: Status message

### Decrypt String
Decrypt a string using symmetric encryption with password-based key derivation.

**Inputs:**
- Ciphertext (required): The encrypted string (base64 encoded)
- Password (required): Password for decryption
- Algorithm: Encryption algorithm used

**Outputs:**
- success: Boolean indicating if decryption succeeded
- plaintext: Decrypted string
- algorithm: Algorithm used
- status: Status message

### Generate Key Pair
Generate an asymmetric key pair (RSA or Ed25519) for signing and encryption operations.

**Inputs:**
- Key Type: Type of key pair (RSA, ED25519)
- Key Size (bits): Key size in bits (RSA: 2048+, Ed25519: fixed)
- Format: Output format (PEM, BASE64)

**Outputs:**
- success: Boolean indicating if generation succeeded
- keyType: Type of key generated
- keySize: Key size in bits
- privateKey: Private key in specified format
- publicKey: Public key in specified format
- format: Format used
- status: Status message

### Sign Data
Sign data using a private key for verification and authentication.

**Inputs:**
- Data (required): The data to sign
- Private Key (required): Private key in PEM or base64 format
- Signature Format: Output format (BASE64, HEX)

**Outputs:**
- success: Boolean indicating if signing succeeded
- signature: Digital signature
- signatureFormat: Format of signature
- status: Status message

### Verify Signature
Verify a signature using a public key to authenticate data integrity.

**Inputs:**
- Data (required): The data that was signed
- Signature (required): The signature to verify
- Public Key (required): Public key in PEM or base64 format
- Signature Format: Format of signature (BASE64, HEX)

**Outputs:**
- success: Boolean indicating if verification completed
- valid: Whether signature is valid
- signatureFormat: Format of signature
- status: Status message

### Derive Key
Derive a cryptographic key from a password using key derivation functions.

**Inputs:**
- Password (required): Password to derive key from
- Salt (required): Salt for key derivation (base64 or "GENERATE")
- Key Length (bytes): Length of derived key (default: 32)
- Algorithm: Key derivation algorithm (PBKDF2, ARGON2)
- Iterations: Number of iterations (default: 100000)
- Output Format: Output format (BASE64, HEX)

**Outputs:**
- success: Boolean indicating if derivation succeeded
- derivedKey: Derived cryptographic key
- salt: Salt used (or generated)
- algorithm: Algorithm used
- keyLength: Key length in bytes
- iterations: Iterations used
- outputFormat: Format of output
- status: Status message

## Encryption Algorithms

### AES-GCM
- Advanced Encryption Standard in Galois/Counter Mode
- Provides both encryption and authentication
- Recommended for most use cases
- 256-bit key size

### ChaCha20-Poly1305
- Stream cipher with authentication
- Faster than AES on systems without AES hardware acceleration
- Excellent mobile/embedded performance
- 256-bit key size

### Fernet
- Symmetric encryption recipe using AES-128-CBC
- Includes timestamp for replay protection
- Simple to use, less configuration
- 128-bit key size

## Key Types

### RSA
- Widely used asymmetric algorithm
- Supports both encryption and signing
- Key sizes: 2048+ bits (2048 recommended, 4096 for high security)
- Slower than Ed25519 but more widely compatible

### Ed25519
- Modern elliptic curve algorithm
- Designed for signing (not encryption)
- Faster and more secure than RSA for same key size
- Smaller keys and signatures
- Recommended for new applications

## Key Derivation Functions

### PBKDF2
- Password-Based Key Derivation Function 2
- Widely supported and battle-tested
- Configurable iterations for security
- SHA-256 hash function
- Good balance of security and performance

### Argon2
- Modern, memory-hard key derivation
- Resistant to GPU/ASIC attacks
- Recommended for new applications
- Higher memory requirements for security
- Best for password-based encryption

## Use Cases

- **File Encryption**: Encrypt sensitive files before storage or transfer
- **Data Protection**: Encrypt strings containing sensitive information
- **Digital Signatures**: Sign documents, API requests, or configuration files
- **Authentication**: Verify authenticity of data or software updates
- **Key Management**: Generate and manage cryptographic keys
- **Password Security**: Derive secure keys from passwords for encryption

## Example Workflows

### Encrypt Sensitive File
```
1. Encrypt File: 
   - filePath="sensitive_data.csv"
   - password="{{vault.password}}"
   - algorithm="AES-GCM"
2. Upload encrypted file to cloud storage
```

### Sign API Request
```
1. Generate Key Pair: keyType="ED25519", format="PEM"
2. Prepare API request data
3. Sign Data: data="{{request_body}}", privateKey="{{1.privateKey}}"
4. Make API request with signature
```

### Verify Downloaded File
```
1. Download file from untrusted source
2. Get public key from trusted source
3. Verify Signature: data="{{file_content}}", signature="{{downloaded_signature}}", publicKey="{{trusted_public_key}}"
4. If valid, process file; if invalid, reject
```

### Secure Password Storage
```
1. Derive Key: 
   - password="{{user_password}}"
   - salt="GENERATE"
   - algorithm="ARGON2"
2. Store derived key and salt in database
3. Use derived key for encryption operations
```

## Security Best Practices

- **Strong Passwords**: Use strong, unique passwords for encryption
- **Key Management**: Never store private keys in plain text
- **Key Rotation**: Regularly rotate encryption keys
- **Salt Usage**: Always use unique salts for key derivation
- **Algorithm Choice**: Use modern algorithms (AES-GCM, Ed25519, Argon2)
- **Key Sizes**: Use recommended minimum key sizes (RSA 2048+, AES 256)
- **Secure Storage**: Store encrypted data in secure locations
- **Backup Keys**: Maintain secure backups of private keys

## Requirements

- cryptography==41.0.7 (included in extension)
- No external software required

## Notes

- All cryptographic operations use the Python cryptography library
- Password-based encryption uses PBKDF2 with 100,000 iterations by default
- RSA signatures use PSS padding with SHA-256
- Ed25519 provides modern, fast digital signatures
- Keys can be generated in PEM or base64 format for flexibility
- File encryption includes metadata (salt, nonce) in the encrypted file
- String encryption outputs base64-encoded ciphertext for easy handling

## Limitations

- Symmetric encryption only (no asymmetric encryption of data)
- Ed25519 keys are for signing only (not encryption)
- Key derivation requires salt management
- No hardware security module (HSM) integration
- No integration with system key stores

## Contract (v1.1.1)

**Install:** Extensions → **Install from file** → choose `crypto-utilities.dsext`.

**Permissions:**
- **filesystem**

See templates/ for the example workflow added on install.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/crypto-utilities
python tools/deskstride_ext_cli.py pack extensions/crypto-utilities -o crypto-utilities.dsext
```
