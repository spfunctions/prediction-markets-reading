# RSA-PSS Authentication

**RSA-PSS (Probabilistic Signature Scheme) is the cryptographic authentication method used by Kalshi's API. Instead of a simple API key, you sign each request with a private key, proving your identity without exposing secrets.**


## Explanation

## How Kalshi Authentication Works

Kalshi uses RSA-PSS key pairs instead of simple API keys. This is more secure because:
- Your private key never leaves your machine
- Each request is signed uniquely (replay attacks are impossible)
- Even if network traffic is intercepted, the attacker can't generate new valid requests

### The Authentication Flow

1. **Generate a key pair**: Create an RSA private key and public key
2. **Upload the public key**: Register your public key with Kalshi
3. **Sign requests**: For each API request, compute an RSA-PSS signature over the request data using your private key
4. **Kalshi verifies**: Kalshi checks the signature against your registered public key

### Setting Up with SimpleFunctions

The CLI handles RSA-PSS complexity for you:

```bash
sf auth add-venue kalshi
```

This interactively walks you through:
1. Generating an RSA key pair (or importing an existing one)
2. Uploading the public key to Kalshi
3. Securely storing the private key on your machine
4. Testing the connection

After setup, every CLI command that touches Kalshi automatically signs requests. You never need to think about RSA-PSS again.

### Security Notes

- Your private key is stored in your OS keychain (macOS Keychain / Linux secret-service)
- The CLI never logs or transmits your private key
- If you suspect key compromise, revoke it on Kalshi immediately and generate a new pair


## Example

# Set up Kalshi authentication
sf auth add-venue kalshi

> Enter your Kalshi email: trader@example.com
> Generating RSA-2048 key pair...
> Public key fingerprint: SHA256:abc123...
> Uploading public key to Kalshi...
> ✓ Key registered successfully
> ✓ Private key stored in system keychain
> ✓ Connection test: OK (200)

# The CLI now signs all Kalshi requests automatically
sf market KXRECSSNBER-26  # ← signed with RSA-PSS under the hood


## CLI

```bash
sf auth add-venue kalshi
```


## Related

[api-key](api-key.md), [cli](cli.md), [rate-limit](rate-limit.md)
