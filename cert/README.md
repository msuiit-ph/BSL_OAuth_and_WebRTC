# How to Generate `cert.pem` and `key.pem` SSL/TLS Certificates

This guide covers generating a self-signed certificate (`cert.pem`) and private key (`key.pem`) using OpenSSL, along with trusted local development and production alternatives.

---

## 1. Quick Start (OpenSSL)

Run the following single command to generate a 4096-bit RSA private key and a self-signed certificate valid for 365 days:

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes -subj "/CN=localhost"
```

### Command Flag Breakdown

| Flag | Description |
| :--- | :--- |
| `req` | Invokes the certificate request and generating utility. |
| `-x509` | Outputs a self-signed certificate instead of a Certificate Signing Request (CSR). |
| `-newkey rsa:4096` | Generates a new 4096-bit RSA private key. |
| `-keyout key.pem` | Saves the generated private key to `key.pem`. |
| `-out cert.pem` | Saves the generated public certificate to `cert.pem`. |
| `-sha256` | Signs the certificate using the SHA-256 hash algorithm. |
| `-days 365` | Sets the validity period of the certificate to 365 days. |
| `-nodes` | ("No DES") Skips encrypting the private key with a passphrase so servers can start automatically without manual password prompts. |
| `-subj "/CN=localhost"` | Sets the Common Name non-interactively (replace `localhost` with your domain or IP). |

---

## 2. Generating Certificates with Subject Alternative Names (SAN)

Modern browsers (Chrome, Firefox, Edge, Safari) require the **Subject Alternative Name (SAN)** extension and will reject certificates that only specify a Common Name (`CN`).

Use `-addext` (OpenSSL 1.1.1+) to include multiple hostnames and IP addresses:

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes \
  -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,DNS:*.localhost,IP:127.0.0.1"
```

---

## 3. Verifying the Generated Files

### Inspect Certificate Details
Check the subject, validity dates, and SAN extensions of `cert.pem`:

```bash
openssl x509 -in cert.pem -text -noout
```

### Verify Key and Certificate Match
Ensure `cert.pem` and `key.pem` belong together (both commands should output identical hashes):

```bash
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
```

---

## 4. Alternatives

### Browser-Trusted Local Development (`mkcert`)
Self-signed certificates trigger browser security warnings (`ERR_CERT_AUTHORITY_INVALID`). To generate locally trusted certificates without manual browser exceptions, use [`mkcert`](https://github.com):

1. **Install local CA (run once):**
   ```bash
   mkcert -install
   ```
2. **Generate `cert.pem` and `key.pem`:**
   ```bash
   mkcert -key-file key.pem -cert-file cert.pem localhost 127.0.0.1 ::1
   ```

### Public Production Websites (Let's Encrypt)
Never use self-signed certificates on public-facing production websites. Use [Let's Encrypt](https://letsencrypt.org) with Certbot to generate free, publicly trusted certificates:

```bash
sudo certbot certonly --standalone -d example.com -d ://example.com
```
*(Certbot outputs `fullchain.pem` and `privkey.pem` in `/etc/letsencrypt/live/://example.com`.)*

---

## 5. Security Best Practices

* **Protect `key.pem`:** Restrict file permissions so only the owner/service account can read the private key:
  ```bash
  chmod 600 key.pem
  ```
* **Never commit `key.pem` to Git:** Add `*.pem` or `key.pem` to your `.gitignore` file immediately to prevent leaking private keys.
