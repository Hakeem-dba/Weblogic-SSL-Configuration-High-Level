# WebLogic / OAS SSL Configuration Guide

## Overview
This repository provides a complete, practical guide for configuring SSL in:

- Oracle WebLogic Server
- Oracle Analytics Server (OAS)

It is designed for:
- DBAs
- Middleware Engineers
- Infrastructure Teams

---

## What You Will Learn

- SSL fundamentals (Identity vs Trust)
- Keystore strategies (JKS vs cacerts)
- SSL with Load Balancers (Offloading / Re-encryption)
- Certificate types (Self-signed vs CA-signed)
- Real-world troubleshooting scenarios

---

## Contents

- `WebLogic-OAS-SSL-Guide.md` → Full technical guide
- Troubleshooting cases included

---

## Use Cases Covered

- Enable HTTPS on WebLogic / OAS
- Configure SSL behind Load Balancer
- Fix SSL handshake errors
- Understand cacerts vs identity keystore
- Prepare production-ready SSL setup

---

## Key Takeaway

> Identity = who the server is  
> Trust = who the server trusts  

---

## Best Practice

- Use Custom Identity Keystore
- Use CA-signed certificates in production
- Avoid modifying Java cacerts unless necessary
- Prefer OHS + Load Balancer architecture for OAS

---

## 👨‍💻 Author

Abed Alhakim Irsheid
