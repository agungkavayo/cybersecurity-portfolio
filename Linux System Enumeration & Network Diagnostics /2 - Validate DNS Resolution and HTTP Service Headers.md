# Validate DNS Resolution and HTTP Service Headers

## Objective

- Verify domain-to-IP resolution using multiple methods  
- Compare DNS resolution paths (system vs direct query)  
- Inspect HTTP response headers from external and local services  
- Identify active services and open ports  
- Understand differences between internal and external enumeration  

## Environment

| System | Role | Description |
|--------|------|------------|
| Kali Linux | Assessment Host | Used for DNS validation, HTTP inspection, and service enumeration |

## Actions Taken

### 1. DNS Resolution

Used two separate approaches to validate domain resolution and compared results:

```bash
dig openai.com +short
dig example.com +short
getent hosts example.com
```

`dig` queries the DNS resolver directly; `getent` uses the OS name resolution stack (including `/etc/hosts`). Differences between the two outputs indicate potential local overrides or split-horizon DNS.

### 2. HTTP Header Inspection

Retrieved response headers from an external target and two local lab services:

```bash
curl -I https://example.com
curl -I http://127.0.0.1:3000
curl -I http://127.0.0.1:8081
```

Analyzed status codes, `Server` headers, redirect behavior, and response timing.

### 3. Service Enumeration — Dual Perspective

```bash
# Internal view
ss -tulpn | head -n 30

# External/assessment view
nmap -Pn -sV -p 22,3000,8081 127.0.0.1
```

## Sample Output

```
HTTP/2 200
content-type: text/html; charset=UTF-8
server: ECAcc (nyd/D10E)
```

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.x
3000/tcp open  http    Node.js Express
```

## Key Finding

`ss` and `nmap` can return different results for the same host. `ss` reflects what the kernel sees; `nmap` reflects what an external assessor sees. This distinction matters in real-world exposure validation.

## What I Learned

A service that appears "closed" from one tool may still be accessible from another perspective. Validating from both sides is essential before concluding that a port is unexposed.
