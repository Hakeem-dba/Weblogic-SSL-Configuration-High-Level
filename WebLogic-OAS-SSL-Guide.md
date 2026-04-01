# WebLogic / OAS SSL Configuration – Complete Practical Guide

---

# Core Concept: Identity vs Trust

## Server Identity = “Who am I?”
Used when clients connect to your server (HTTPS)

Contains:
- Private key
- Server certificate
- Certificate chain

👉 Required to enable HTTPS

---

## Trust = “Who do I trust?”
Used when WebLogic connects to other systems

Contains:
- CA certificates only

---

## Key Rule

| Component | Purpose |
|----------|--------|
| Identity Keystore | Server proves its identity |
| Truststore / cacerts | Server verifies others |

---

# Keystore Options

## Demo Identity & Demo Trust
- Self-signed
- Dev only

---

## Custom Identity + Custom Trust (Best Practice)

---

## Custom Identity + Java Standard Trust (cacerts)

---

# cacerts Explained

Location:
$JAVA_HOME/lib/security/cacerts

✔ Used for trust  
❌ Not used for identity  

---

# JKS vs cacerts

| Option | Use |
|------|-----|
| JKS / PKCS12 | Identity + Trust (preferred) |
| cacerts | Trust only |

---

# Load Balancer Scenarios

## SSL Offloading
Client → HTTPS → LB → HTTP → Backend

---

## SSL Re-encryption
Client → HTTPS → LB → HTTPS → Backend

---

# Enable HTTPS (Recommended)

## Self-Signed Example

keytool -genkeypair \
 -alias oas_ssl \
 -keyalg RSA \
 -keystore identity.jks \
 -validity 365 \
 -dname "CN=analytics.company.com"

---

# Certificate Types

| Type | Use |
|------|-----|
| Self-signed | Internal / backend |
| Internal CA | Enterprise |
| Public CA | External users |

---

# Common Mistakes

- Importing into cacerts for HTTPS
- CN mismatch
- Missing chain
- Using demo in production

---

# TROUBLESHOOTING (REAL CASES)

---

## 🔴 Case 1: SSLHandshakeException (PKIX path building failed)

### Error:
javax.net.ssl.SSLHandshakeException: PKIX path building failed

### Cause:
WebLogic does not trust remote certificate

### Fix:
Import certificate into truststore:

keytool -import -alias api_cert \
 -file api.crt \
 -keystore cacerts

---

## 🔴 Case 2: HTTPS works but browser shows "Not Secure"

### Cause:
- Self-signed certificate
- Missing intermediate cert

### Fix:
- Use CA-signed certificate OR
- Import CA into browser

---

## 🔴 Case 3: Hostname verification failed

### Error:
hostname verification failed

### Cause:
Certificate CN ≠ URL

Example:
URL: analytics.company.com  
Cert: server01  

### Fix:
Regenerate cert with correct CN/SAN

---

## 🔴 Case 4: LB cannot connect to backend HTTPS

### Cause:
- Self-signed cert not trusted
- Hostname mismatch
- Strict LB validation

### Fix:
- Disable validation OR
- Use internal CA cert

---

## 🔴 Case 5: Redirects go to HTTP instead of HTTPS

### Cause:
WebLogic not aware of proxy

### Fix:
Enable:
- WebLogic Plug-In Enabled

---

## 🔴 Case 6: HTTPS enabled but port not accessible

### Cause:
- SSL port not enabled
- Firewall issue

### Fix:
- Enable SSL port in WebLogic
- Check network/firewall

---

## 🔴 Case 7: "No identity key/certificate configured"

### Cause:
Identity keystore missing

### Fix:
Configure:
- Identity keystore
- Private key alias

---

## 🔴 Case 8: Import to cacerts but SSL still fails

### Cause:
- Wrong certificate imported
- Missing chain

### Fix:
Import full chain (root + intermediate)

---

# 📋 What to Ask Before SSL Setup

- Certificate type?
- Final URL?
- LB validation?
- SSL scope?
- OHS or direct WebLogic?

---

# Final Recommendations

## Production
- Custom Identity
- CA-signed cert
- OHS + LB

---

## LB Backend HTTPS Only
- Use self-signed or internal cert
- Avoid demo identity
- Confirm LB behavior

---

# 🧩 Final Summary

- Identity = who the server is
- Trust = who the server trusts
- cacerts = trust only
- HTTPS requires identity
