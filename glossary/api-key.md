# API Key

**An API key is a secret token that authenticates your requests to SimpleFunctions and to prediction market venues (Kalshi, Polymarket). Each key has permissions and rate limits.**


## Explanation

## API Key Management

API keys are how you prove your identity when making programmatic requests. SimpleFunctions uses API keys at two levels:

### SimpleFunctions API Key

Your SF API key (`sf_live_...`) authenticates requests to the SimpleFunctions platform:
- Thesis management
- Edge calculations
- Portfolio data
- Heartbeat monitoring

### Venue API Keys

To trade or access real-time data, you also need keys from the venues themselves:
- **Kalshi**: Uses RSA-PSS key pairs for authentication (more complex but more secure)
- **Polymarket**: Uses wallet-based authentication

### Security Best Practices

1. **Never commit keys to git**: Use environment variables or `.env.local`
2. **Use key rotation**: Revoke and regenerate keys periodically
3. **Scope permissions**: Create read-only keys for monitoring, read-write for trading
4. **Monitor usage**: Check `sf keys status` for last-used timestamps

### Setting Up Keys

```bash
sf auth login              # Authenticate with SimpleFunctions
sf auth add-venue kalshi   # Add Kalshi credentials
sf auth add-venue polymarket # Add Polymarket credentials
```

The CLI securely stores venue credentials and handles authentication headers automatically. You never need to manually construct authentication headers.


## Example

# Generate a new API key
sf keys create --name "production-bot"
> Created key: sf_live_abc123...
> Key prefix: sf_live_abc1 (for identification)

# List your keys
sf keys list
Name              Prefix          Created      Last Used
production-bot    sf_live_abc1    2026-01-15   2 hours ago
read-only         sf_live_def4    2026-02-01   1 day ago

# Revoke a key
sf keys revoke sf_live_abc1

# Environment variable setup
export SF_API_KEY=sf_live_abc123...


## CLI

```bash
sf keys
```


## Related

[rsa-pss](rsa-pss.md), [cli](cli.md), [rate-limit](rate-limit.md), [webhook](webhook.md)
