# Moving Domain DNS Management to AWS Route53

A common scenario is:

1. Domain purchased from Hostinger or GoDaddy.
2. DNS management needs to be handled by AWS Route53.

Example:

```text
Domain Registrar : Hostinger
Domain Name      : daws86s.fun

DNS Management   : AWS Route53
```

---

# Why Move DNS Management to AWS?

Advantages:

- Better AWS Integration
- Easy Management
- Infrastructure Automation
- High Availability
- Route53 Health Checks
- Route53 Routing Policies

---

# Current Ownership vs DNS Authority

Purchasing a domain and managing DNS are two different things.

Example:

```text
Domain Owner:
Hostinger

DNS Authority:
AWS Route53
```

Think of it as:

```text
Domain Registration = Ownership

Nameservers = Authority to Manage DNS
```

---

# Step 1: Purchase Domain

Purchase domain from:

- Hostinger
- GoDaddy
- Namecheap

Example:

```text
daws86s.fun
```

At this stage:

```text
Registrar = Hostinger
DNS Authority = Hostinger
```

---

# Step 2: Create Hosted Zone in Route53

Open AWS Console:

```text
Route53
```

Create Hosted Zone:

```text
daws86s.fun
```

AWS automatically creates:

```text
NS Records
SOA Record
```

Example:

```text
ns-123.awsdns-10.com
ns-456.awsdns-20.net
ns-789.awsdns-30.org
ns-321.awsdns-40.co.uk
```

These are AWS Route53 Nameservers.

---

# Step 3: Copy Route53 Nameservers

Example:

```text
ns-123.awsdns-10.com
ns-456.awsdns-20.net
ns-789.awsdns-30.org
ns-321.awsdns-40.co.uk
```

Keep these values.

---

# Step 4: Login to Registrar

Example:

```text
Hostinger
```

Navigate:

```text
Domains
  └── Manage Domain
         └── Nameservers
```

---

# Step 5: Replace Existing Nameservers

Current:

```text
ns1.hostinger.com
ns2.hostinger.com
```

Replace with:

```text
ns-123.awsdns-10.com
ns-456.awsdns-20.net
ns-789.awsdns-30.org
ns-321.awsdns-40.co.uk
```

Save changes.

---

# Step 6: Wait for DNS Propagation

DNS propagation usually takes:

```text
Few Minutes
to
24-48 Hours
```

depending on caching.

---

# Step 7: Verify Route53 is Authoritative

Check:

```bash
nslookup daws86s.fun
```

or

```bash
dig NS daws86s.fun
```

Expected Output:

```text
AWS Route53 Nameservers
```

---

# Step 8: Create DNS Records in Route53

Example:

## Frontend

```text
daws86s.fun
      │
      ▼
54.88.122.243
```

Create:

```text
A Record
```

---

## Database

```text
mysql.daws86s.fun
      │
      ▼
172.31.30.95
```

Create:

```text
A Record
```

---

# Final Architecture

```text
User
  │
  ▼
DNS Query
  │
  ▼
AWS Route53
  │
  ▼
A Record
  │
  ▼
Public IP
  │
  ▼
Nginx Frontend
```

---

# Important Interview Question

## Does Domain Ownership Move to AWS?

Answer:

```text
No
```

Example:

```text
Registrar      : Hostinger

DNS Authority  : AWS Route53
```

The domain is still owned through Hostinger.

Only DNS management is delegated to AWS Route53.

---

# Key Concept

```text
Domain Purchase
        ≠
DNS Management
```

You can:

```text
Buy Domain from Hostinger

Manage DNS from AWS Route53
```

simply by updating Nameservers.