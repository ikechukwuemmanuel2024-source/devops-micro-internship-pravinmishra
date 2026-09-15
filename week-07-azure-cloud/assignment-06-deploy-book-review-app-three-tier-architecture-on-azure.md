# Assignment 6 — Capstone: Deploy Book Review App (Three-Tier Architecture) on Azure

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a production-ready, best-practice-compliant three-tier architecture on Azure: separated presentation, application, and database tiers, least-privilege network access, a controlled public entry point, protected secrets, and availability/monitoring evidence.

---

# Task 1 — Design the Azure Three-Tier Architecture

## Goal

Create an architecture diagram and implementation plan identifying the presentation, application, and database components, the chosen Azure services, the public entry point, and the internal traffic paths.

### Evidence

#### Screenshot 1 — Architecture diagram showing the public entry point, three tiers, network boundaries, and traffic flow

![Screenshot 0](screenshots/week7-ass6-00.png)

---

#### Screenshot 2 — Written architecture assumptions and selected Azure services

![Screenshot 0](screenshots/week7-ass6-01.png)

---

# Task 2 — Create the Azure Network Foundation

## Goal

Create a dedicated Resource Group and VNet with separate subnets for the web, application, and database tiers, keeping the application and database tiers without direct public access.

### Evidence

#### Screenshot 3 — Resource Group overview showing the assignment resources

![Screenshot 0](screenshots/week7-ass6-02.png)

---

#### Screenshot 4 — VNet overview showing the address space and all required subnets

![Screenshot 0](screenshots/week7-ass6-03.png)

---

#### Screenshot 5 — Route-table or Private DNS evidence where applicable

![Screenshot 0](screenshots/week7-ass6-04.png)

---

# Task 3 — Configure Security and Secret Management

## Goal

Apply least-privilege NSG rules so traffic flows Internet → public entry point → web tier → application tier → database tier, and store credentials in Azure Key Vault or another approved secure mechanism.

### Evidence

#### Screenshot 6 — NSG rules proving least-privilege access between the tiers

![Screenshot 0](screenshots/week7-ass6-17.png)

---

#### Screenshot 7 — Key Vault or approved secret-management configuration (without displaying secret values)

![Screenshot 0](screenshots/week7-ass6-05.png)

---

# Task 4 — Deploy the Presentation (Web) Tier

## Goal

Deploy the Book Review App presentation layer on the approved web-tier compute service, configured to route requests to the internal application-tier endpoint, and not directly exposed except through the public entry service.

### Evidence

#### Screenshot 8 — Web-tier compute overview showing subnet and availability configuration

![Screenshot 0](screenshots/week7-ass6-06.png)

---

#### Screenshot 9 — Terminal or service output proving the presentation layer is running

![Screenshot 0](screenshots/week7-ass6-07.png)

---

# Task 5 — Deploy the Business (Application) Tier

## Goal

Deploy the Book Review App backend privately in the application subnet, configured to use the private database endpoint and secured environment values, reachable only through its internal endpoint.

### Evidence

#### Screenshot 10 — Application-tier compute overview showing private subnet placement

![Screenshot 0](screenshots/week7-ass6-08.png)

---

#### Screenshot 11 — Backend process, service, or listening-port evidence

![Screenshot 0](screenshots/week7-ass6-09.png)

---

#### Screenshot 12 — Internal health-check or API response (without exposing secrets)

![Screenshot 0](screenshots/week7-ass6-10.png)

---

# Task 6 — Deploy the Managed Database Tier

## Goal

Create a private Azure managed database (public access disabled), with availability/backup/retention settings, the Book Review App schema imported, and access restricted to the application tier only.

### Evidence

#### Screenshot 13 — Database overview showing private connectivity and public access disabled

![Screenshot 0](screenshots/week7-ass6-11.png)

---

#### Screenshot 14 — Availability, backup, and retention configuration

![Screenshot 0](screenshots/week7-ass6-12.png)

---

#### Screenshot 15 — Successful schema or connectivity verification (without exposing credentials)

![Screenshot 0](screenshots/week7-ass6-13.png)

---

# Task 7 — Configure Traffic Management, Availability, and Monitoring

## Goal

Configure the approved public entry service with health probes and backend pools, internal routing for the application tier where required, and enable Azure Monitor/diagnostics/logs/alerts for the key resources.

### Evidence

#### Screenshot 16 — Public entry service showing listener, frontend endpoint, and healthy web targets

![Screenshot 0](screenshots/week7-ass6-14.png)

---

#### Screenshot 17 — Internal application-tier load-balancing or routing configuration where applicable

![Screenshot 0](screenshots/week7-ass6-15.png)

---

#### Screenshot 18 — Azure Monitor, diagnostic settings, logs, metrics, or alert evidence

![Screenshot 0](screenshots/week7-ass6-16.png)

---

# Task 8 — Validate the Production-Style Deployment

## Goal

Confirm the Book Review App works end to end through the public endpoint, with at least one database read and one write, confirm private tiers are not internet-reachable, and complete a safe availability test.

### Evidence

#### Screenshot 19 — Browser showing the Book Review App through the public endpoint

![Screenshot 0](screenshots/week7-ass6-18.png)

---

#### Screenshot 20 — Proof of successful database-backed read and write operations

![Screenshot 0](screenshots/week7-ass6-19.png)

---

#### Screenshot 21 — Evidence that private tiers are not publicly accessible

![Screenshot 0](screenshots/week7-ass6-20.png)

---

#### Screenshot 22 — Availability-test and healthy-target evidence

![Screenshot 0](screenshots/week7-ass6-21.png)

---

#### Public Endpoint

Paste your public endpoint URL here:

`http://http://4.253.20.120`

---

### Notes

Summarize what worked, issues encountered and how they were fixed, and the availability/security/secrets/monitoring/backup choices made.

# Assignment 6 — Deployment Summary

## 1. What Worked

The three-tier Book Review application was successfully deployed on Microsoft Azure.

The final architecture consists of:

**Internet → Azure Load Balancer → Web VM/Nginx → App VM/EpicBook → Private MySQL Flexible Server**

The following components were successfully configured and tested:

- Azure Virtual Network: `book-review-vnet`
- Web subnet: `10.0.1.0/24`
- App subnet: `10.0.2.0/24`
- Database subnet: `10.0.3.0/24`
- Web VM running Nginx
- App VM running the EpicBook Node.js application on port `8080`
- Azure MySQL Flexible Server using private network access
- Azure Load Balancer providing the public entry point
- Azure Key Vault storing application secrets
- Network Security Groups controlling traffic between tiers
- Azure Monitor CPU alert for monitoring

The application was successfully accessed through the public Load Balancer endpoint:

**`http://4.253.20.120`**

The database was also successfully accessed privately from the application tier, and database read/write operations were validated.

---

## 2. Issues Encountered and Fixes

### Application was initially not reachable through the Load Balancer

The Load Balancer initially returned connection failures and later a `502` error.

The issue was traced to the application on the App VM not being available on port `8080`.

**Fix:** The EpicBook application was restarted with:

```bash
npm run start
```

After the application was running again, Nginx on the Web VM could proxy requests to:

```text
10.0.2.4:8080
```

The public Load Balancer endpoint then successfully displayed the EpicBook website.

### Load Balancer rule had an incorrect backend port

The Load Balancer rule initially had an incorrect backend port configuration.

**Fix:** The rule was corrected to:

```text
TCP 80 → TCP 80
```

The HTTP health probe was also configured on port `80`.

### SSH security was initially too broad

The Web NSG initially allowed SSH access more broadly than necessary.

**Fix:** SSH access was restricted to the user's public IP:

```text
129.222.206.129
```

The same principle was applied to the App VM's SSH rule.

### Application-to-database connectivity

The application needed to communicate with the private MySQL server.

**Fix:** The database was configured for private VNet access, and the application was configured to use the database's private IP:

```text
10.0.3.4
```

Connectivity was successfully verified from the App VM.

---

## 3. Availability Choices

An Azure **Standard Public Load Balancer** was used as the public entry point.

Configuration included:

- Standard SKU
- Regional load balancer
- Zone-redundant frontend
- Public frontend IP: `4.253.20.120`
- Backend pool containing the Web VM
- HTTP health probe on port `80`
- Load-balancing rule on TCP port `80`

The health probe allows Azure to determine whether the Web VM is available before sending traffic to it.

The application was also validated through the public endpoint and the Load Balancer backend health status.

---

## 4. Security Choices

The architecture follows a three-tier security model.

### Web tier

The Web VM is responsible for receiving web traffic through Nginx.

The Web NSG allows:

- HTTP — port `80`
- HTTPS — port `443`
- SSH — port `22`, restricted to the administrator's IP

### Application tier

The App VM is not intended to be directly accessible from the Internet on its application port.

Port `8080` is allowed only from the Web subnet:

```text
10.0.1.0/24 → 10.0.2.4:8080
```

### Database tier

MySQL port `3306` is restricted to the application subnet:

```text
10.0.2.0/24 → MySQL:3306
```

The MySQL server has **public access disabled** and uses private VNet connectivity.

Internet tests confirmed that the application and database ports were not directly reachable from the Internet.

---

## 5. Secret Management

Azure Key Vault was used to avoid relying solely on hard-coded credentials.

The Key Vault contains secrets for:

- MySQL administrator password
- MySQL server hostname
- Database name

The actual password value was not exposed in the architecture documentation or screenshots.

The database itself uses MySQL authentication, while network-level access is restricted through the private VNet and NSG rules.

---

## 6. Monitoring

Azure Monitor was configured with a CPU alert:

**`book-review-web-high-cpu-alert`**

The alert monitors CPU usage and triggers when average CPU exceeds:

```text
80%
```

over a five-minute period.

An Action Group was configured to provide email notification when the alert condition occurs.

This provides basic operational monitoring and allows high CPU usage to be detected without manually checking the VM.

---

## 7. Backup and Database Reliability

The MySQL Flexible Server was configured with:

- **7-day backup retention**
- **24-hour backup interval**
- **Locally redundant backup storage**
- Geo-redundant backup disabled
- High availability disabled

The database uses Azure Database for MySQL Flexible Server rather than a self-managed MySQL installation, allowing Azure to handle the managed database backup functionality.

The database was also successfully tested for read/write operations using the `bookstore` database.

---

## 8. Final Validation

The final validation demonstrated that:

1. The public Load Balancer endpoint successfully served the EpicBook website.
2. The App VM could communicate with the private MySQL server.
3. Database tables and book records could be read.
4. A test record could be written to the database and read back.
5. The test table was removed after validation.
6. The App VM's application port was not directly reachable from the Internet.
7. The private MySQL port was not directly reachable from the Internet.
8. The Load Balancer reported the Web VM as healthy.

Overall, the deployment achieved the required three-tier architecture with **public access limited to the entry point, private application/database tiers, controlled network access, secret management, monitoring, backups, and end-to-end validation**.

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, keys, connection strings, or subscription IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram and assumptions documented (Screenshots 1–2)
- [ ] Task 2: Network foundation created with isolated tiers (Screenshots 3–5)
- [ ] Task 3: Least-privilege security and secret management configured (Screenshots 6–7)
- [ ] Task 4: Presentation tier deployed (Screenshots 8–9)
- [ ] Task 5: Application tier deployed privately (Screenshots 10–12)
- [ ] Task 6: Managed database tier deployed privately (Screenshots 13–15)
- [ ] Task 7: Public entry, internal routing, and monitoring configured (Screenshots 16–18)
- [ ] Task 8: End-to-end validation and availability test completed (Screenshots 19–22, Public Endpoint, Notes)
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
