# Security Notes

## Documented Vulnerabilities

The BlockmanGO API has several security weaknesses identified during reverse engineering:

### 1. No Certificate Pinning

The API does not implement SSL certificate pinning, allowing MITM interception of all traffic using tools like mitmproxy.

### 2. MD5 Request Signing

The `x-sign` header uses MD5, which is cryptographically broken. The signing scheme is fully reproducible and can be implemented by anyone with the API keys.

### 3. Static API Keys

Three API key pairs are hardcoded in the APK and never rotate. They are:
- `6aDtpIdzQdgGwrpP6HzuPA` / `9EuDKGtoWAOWoQH1cRng-d5ihNN60hkGLaRiaZTk-6s`
- `h0jCHbhVd9Fpkx-FGkxeRw` / `lOTB7DNdMMpdyUO-psJ5b2ivYGmU5RAy6j6bkpoMYcs`
- `dM9XM3sxjfVI6AC77GS9rw` / `6aNQVhd8pP-Gg7_xM2PTEp92G-77tzHGnPKrwslxmAg`

### 4. User IDs Exposed as Plain Integers

User IDs are transmitted as plain integers in the `userid` header. They are sequential and enumerable.

### 5. User Stats Fully Enumerable

The `targetId` parameter on several endpoints allows querying any user's stats, highlights, and profile data without authorization.

### 6. Game Server IPs Leaked

Game server IP addresses and ports are exposed in auth responses and room lists:
- `43.166.142.211:31952`
- `43.130.90.110:7461`
- `43.172.5.8:7014`
- And others

### 7. Mining Pool Values Exposed

Mining room responses include `totalPoolValue`, `remainingPoolValue`, and `miningRewardBonus` — internal economic data.

### 8. Analytics Over HTTP

Analytics events are sent to `http://di.sandboxol.com` (not HTTPS), exposing user behavior data.

### 9. No Rate Limiting Observed

No rate limiting was observed during testing, allowing bulk enumeration of user data.

### 10. RSA Key Size

Password encryption uses a 1024-bit RSA key, which is below the modern minimum of 2048 bits.

## API Key Rotation

The API keys appear to be permanent. The same keys have been observed across multiple app versions and time periods.

## Recommended Defenses (for SandboxOL)

- Implement certificate pinning
- Rotate API keys regularly
- Use SHA-256 or HMAC for request signing
- Add rate limiting
- Migrate analytics to HTTPS
- Upgrade RSA key size to 2048+ bits
- Add request nonce validation to prevent replay attacks

## Disclaimer

This security analysis is provided for educational purposes. Do not use these findings to harm the game, its players, or its infrastructure. Respect the game's Terms of Service.
