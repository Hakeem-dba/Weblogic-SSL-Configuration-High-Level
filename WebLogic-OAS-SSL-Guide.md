# WebLogic / OAS SSL Configuration – Complete Guide

---

# Overview

This guide provides a complete, practical, and production-oriented explanation of SSL configuration in:

- Oracle WebLogic Server
- Oracle Analytics Server (OAS)

It covers:
- Core SSL concepts (Identity vs Trust)
- Keystore strategies (JKS, PKCS12, cacerts)
- Load Balancer SSL scenarios (Offloading & Re-encryption)
- Certificate types and when to use each
- Real-world troubleshooting cases

---

# 1. Core Concept: Identity vs Trust

Understanding this section is critical — everything in SSL depends on it.

---

## 1.1 Server Identity = "Who am I?"

This is what your server uses to **identify itself to clients**.

### Used when:
- A browser accesses OAS
- A user connects to WebLogic console
- Any inbound HTTPS connection

### Contains:
- Private Key (VERY IMPORTANT)
- Server Certificate
- Certificate Chain (Intermediate + Root)

### Stored in:
- Identity Keystore (JKS or PKCS12)

---

## 1.2 Trust = "Who do I trust?"

This is used when your server connects to other systems.

### Used when:
- WebLogic calls external APIs
- OAS connects to DB using SSL
- Any outbound HTTPS call

### Contains:
- Trusted CA certificates only
- NO private key

### Stored in:
- Trust Keystore
- OR Java default: cacerts

---

## Golden Rule

| Component | Purpose |
|----------|--------|
| Identity Keystore | Server proves its identity |
| Truststore / cacerts | Server verifies others |

---

# 2. Keystore Types & Options

---

## 2.1 Demo Identity & Demo Trust

### Description:
- Default WebLogic configuration
- Uses self-signed certificates

### Use:
- Development only

### Limitations:
- Wrong hostname
- Not trusted
- Not suitable for production

---

## 2.2 Custom Identity + Custom Trust (Best Practice)

### Description:
- Full control over certificates
- Separate keystores for identity and trust

### Benefits:
- Environment isolation (DEV / UAT / PROD)
- Better security
- Easier troubleshooting

---

## 2.3 Custom Identity + Java Standard Trust

### Description:
- Identity in custom keystore
- Trust uses Java cacerts

### When to use:
- When system already trusts required CAs

---

# 3. Understanding cacerts

---

## Location:
```

$JAVA_HOME/lib/security/cacerts

```

---

## What it does:
- Stores trusted CA certificates
- Used for outbound SSL

---

## What it DOES NOT do:
- Does NOT store private keys
- Does NOT provide server identity
- Cannot enable HTTPS alone

---

# 4. JKS vs PKCS12 vs cacerts

---

## JKS (Java Keystore)
- Traditional Java format
- Common in older environments

---

## PKCS12 (.p12 / .pfx)
- Modern standard
- Preferred for interoperability
- Usually provided by security teams

---

## cacerts
- Java default truststore
- Shared across all Java applications

---

## Recommendation

| Use Case | Recommended |
|--------|------------|
| Identity | PKCS12 or JKS |
| Trust | Custom truststore or cacerts |
| Production | Avoid modifying cacerts |

---

# 5. SSL with Load Balancer

---

## 5.1 SSL Offloading

### Architecture:
```

Client → HTTPS → Load Balancer → HTTP → Backend

```

### Characteristics:
- SSL terminates at LB
- Backend is not encrypted

---

## 5.2 SSL Offloading + Re-encryption

### Architecture:
```

Client → HTTPS → LB → HTTPS → Backend

```

### Characteristics:
- LB decrypts and re-encrypts
- Backend must support HTTPS

👉 This is very common in enterprise setups

---

# 6. Best Architecture for OAS

```

Client → LB (HTTPS)
→ OHS (HTTPS)
→ WebLogic/OAS (HTTP or HTTPS)

```

### Why:
- Standard Oracle architecture
- Better scalability
- Easier certificate management

---

# 7. How to Enable HTTPS

---

## 7.1 Using Demo Identity (Quick)

- Enable SSL in WebLogic
- Uses demo certificate

⚠️ Not recommended for production

---

## 7.2 Using Self-Signed Certificate (Recommended for Backend)

### Generate certificate:

```

keytool -genkeypair 
-alias oas_ssl 
-keyalg RSA 
-keystore identity.jks 
-validity 365 
-dname "CN=analytics.company.com, OU=IT, O=Company, L=AbuDhabi, C=AE"

```

---

## 7.3 Using CA-Signed Certificate (Production)

Steps:
1. Generate CSR
2. Send to CA
3. Import certificate + chain
4. Configure keystore

---

# 8. Certificate Types

---

## 8.1 Self-Signed

### Use:
- Development
- Internal backend (LB → server)

---

## 8.2 Internal CA

### Use:
- Enterprise internal systems
- Corporate environments

---

## 8.3 Public CA

### Use:
- Internet-facing systems
- External users

---

# 9. Common Mistakes

---

## ❌ Importing certificate into cacerts to enable HTTPS
→ Only affects trust

---

## ❌ CN mismatch

Example:
```

URL: analytics.company.com
Cert: server01

```

→ SSL failure

---

## ❌ Missing intermediate certificate
→ Browser shows warning

---

## ❌ Using Demo Identity in production
→ Security risk

---

# 10. WebLogic Behind Load Balancer / OHS

---

## Important Setting:

Enable:
- WebLogic Plug-In Enabled

---

## Why:
- Ensures correct HTTPS URLs
- Prevents redirect issues

---

# 11. REAL TROUBLESHOOTING CASES

---

## 🔴 Case 1: PKIX Path Building Failed

### Error:
```

javax.net.ssl.SSLHandshakeException:
PKIX path building failed

```

### Cause:
- Target certificate not trusted

### Fix:
```

keytool -import -alias api_cert 
-file api.crt 
-keystore cacerts

```

---

## 🔴 Case 2: Browser shows "Not Secure"

### Cause:
- Self-signed certificate
- Missing chain

### Fix:
- Use CA-signed cert OR
- Import CA into browser

---

## 🔴 Case 3: Hostname Verification Failed

### Cause:
- CN does not match URL

### Fix:
- Regenerate certificate with correct CN/SAN

---

## 🔴 Case 4: Load Balancer Cannot Connect

### Cause:
- Backend cert not trusted
- Strict validation

### Fix:
- Use internal CA OR
- Adjust LB validation

---

## 🔴 Case 5: Redirect goes to HTTP

### Cause:
- WebLogic not aware of HTTPS

### Fix:
- Enable WebLogic Plug-In Enabled

---

## 🔴 Case 6: HTTPS Port Not Accessible

### Cause:
- SSL not enabled
- Firewall blocked

---

## 🔴 Case 7: No Identity Configured

### Error:
```

No identity key/certificate configured

```

### Fix:
- Configure identity keystore
- Set alias and password

---

## 🔴 Case 8: Imported into cacerts but still failing

### Cause:
- Missing intermediate cert

### Fix:
- Import full chain

---

# 12. What to Ask Before SSL Implementation

- Certificate type?
- Final URL?
- Load balancer behavior?
- Backend HTTPS required?
- OHS or direct WebLogic?

---

# 13. Final Recommendations

---

## Production:
- Custom Identity Keystore
- CA-signed certificate
- OHS + Load Balancer

---

## Backend HTTPS only:
- Use self-signed or internal cert
- Avoid demo identity
- Confirm LB behavior

