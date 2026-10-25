# Assignment: Build and Support a 20-User MedSecure Company

**Owner:** Sameer Raut  
**Platform:** Local Windows host with Oracle VirtualBox  
**Status:** Proposed build; implementation and verification pending  
**Scope:** A new local environment. The eduLAB Ubuntu server is outside this lab.

## 1. Business brief

You are the IT administrator for a small healthcare services company with 20 employees. Staff need reliable access to MedSecure, shared files, a CRM, printing and remote access. Your physical Windows PC is your admin desk and represents an offsite administrator during VPN exercises.

Build the environment in stages, explain the purpose of each service, demonstrate normal operation, deliberately introduce controlled faults, and document how you diagnose and resolve them. Use synthetic employee, patient and customer data.

Twenty employees means twenty identities and representative client machines, not twenty simultaneously running VMs. Initially validate individual workflows; simultaneous load testing is a separate task.

## 2. Learning outcomes

By completion, demonstrate that you can:

- Explain the path from a workstation through networking, DNS, authentication and the web server to its database.
- Administer Windows servers remotely with separate admin credentials.
- Manage joiners, movers and leavers using AD groups and application roles.
- Support Windows and Linux clients, with macOS added when suitable hardware is available.
- Investigate DHCP, DNS, permissions, services, VPN and printer faults.
- Explain database tables, relationships, migrations, access controls and restore procedures.
- Distinguish MedSecure clinical workflows from CRM customer-management workflows.
- Produce concise GitHub evidence that distinguishes planned work from tested results.

## 3. Architecture

### Stage A: Prove the application

Physical Windows host -> host-only network -> APP01 Windows Server 2022 -> MedSecure -> its current database.

Start with the existing server as APP01 if it is not already a domain controller. If it already hosts AD, retain it as DC01 and create a separate APP01. Inspect existing roles and take a snapshot before changing anything.

Use RDP from the host for administration and a browser on the host for MedSecure. Do not change the host's normal home-network configuration or join the physical host to the lab domain.

### Stage B: Add company services

| Device | Role | Proposed company LAN address |
|---|---|---|
| DC01 | AD DS, DNS and DHCP | 192.168.56.10 |
| APP01 | Windows Server 2022, MedSecure, application logs | 192.168.56.20 |
| FILE01 | Member server for departmental shares and print queues | 192.168.56.30 |
| WINCLIENT01 | Domain-joined Windows Pro/Enterprise test workstation | DHCP |
| LINUXCLIENT01 | Linux Mint desktop test workstation | DHCP |
| CRM01 | Separate CRM service, introduced after application discovery | 192.168.56.40 |
| DB01 | Optional separate database service after compatibility review | 192.168.56.50 |
| Physical Windows host | Admin workstation outside the lab domain | Host-only interface 192.168.56.1 initially |

Address plan is provisional. Check conflicts with home LANs, existing adapters and VPNs before applying it. Servers use static addresses outside the DHCP pool. Proposed domain: company.test; proposed app name: medsecure.company.test.

The company LAN is one subnet initially. VLANs, additional subnets and routing are later extensions, not prerequisites.

### Stage C: Simulate a remote office connection

Add a firewall/router VPN VM with three interfaces:

| Interface | VirtualBox mode | Example address | Purpose |
|---|---|---|---|
| WAN | NAT | Supplied by VirtualBox | Outbound updates |
| Company LAN | Internal Network named COMPANY-LAN | 192.168.56.254/24 | Gateway for company VMs |
| Remote transit | Host-only network | 192.168.57.254/24 | VPN endpoint reachable by physical host |

In Stage C, move company VMs onto COMPANY-LAN and remove the physical host's direct company-LAN adapter. The host retains only a remote-transit adapter, proposed address 192.168.57.1/24 with no default gateway. Confirm no other direct route to the company LAN remains.

Proposed VPN address pool: 10.77.0.0/24. Allow the VPN handshake from remote transit; deny ordinary transit-to-company forwarding. Permit authenticated VPN clients to reach only the approved company services. Configure return routing and distribute internal DNS through the VPN.

The host keeps its normal internet connection. It establishes a VPN to 192.168.57.254, then reaches company services through the tunnel. This is an offsite simulation on one physical PC, not a public internet deployment.

Do not expose RDP directly to the internet. Disable VirtualBox DHCP on lab segments where Windows DHCP is authoritative. Bind Windows DHCP to the company interface.

### Remote desktop versus application access

- Administrator: host -> VPN when introduced -> RDP to authorised servers.
- Employees: client browser -> MedSecure/CRM; client -> file and print services.
- Employees do not all need remote desktops on the server.
- Administrative RDP is not a 20-user desktop platform. A shared-desktop deployment would require a separate RDS design and licensing review.

## 4. Company identity model

| Department | Employees | Example resources |
|---|---|---|
| Clinical | 8 | MedSecure clinical role and Clinical share |
| Administration | 5 | Approved appointment/admin workflows and Admin share |
| Finance | 3 | Finance share and approved billing workflow |
| Management | 2 | Approved reports and Management share |
| IT | 2 | Support tasks; separate privileged admin accounts |
| Total | 20 | |

Create department OUs, security groups and named test accounts. Grant file permissions through resource groups rather than individual users. Use separate read and modify permissions.

Maintain a mapping between AD groups and application roles. AD login does not automatically provide MedSecure SSO or patient access. Inspect existing authentication first; document local login, SSO and MFA independently. Preserve any working integration and test authorisation separately.

## 5. Assignment tasks and acceptance criteria

| Phase | Work to complete and explain | Acceptance evidence |
|---|---|---|
| 0: Discovery | Record host RAM, CPU, free storage, VirtualBox version, VM roles and current addressing. Locate the actual MedSecure source, launch method and dependencies. | Inventory, resource budget, baseline snapshot and build checklist |
| 1: Connectivity and RDP | Configure the initial host-only network, static APP01 address and scoped firewall rules. Enable admin RDP with NLA. | Host reaches APP01; approved admin login succeeds; unrelated ports remain closed |
| 2: MedSecure | Create its Python environment; verify Windows compatibility; run with an appropriate Windows WSGI server, such as Waitress. Configure logs and eventual service startup. | Browser login, role tests, process/port evidence and restart test |
| 3: Database discovery | Identify the actual engine, connection configuration, schema, migrations, data location and backup method. | Sanitised schema/ER diagram, synthetic record flow and successful restore |
| 4: Identity and DNS | Deploy or inspect DC01, create the domain, 20 accounts, groups and DNS records. Join a Windows client. | Domain login, name resolution, group access tests and joiner/mover/leaver evidence |
| 5: DHCP and clients | Set a lab DHCP scope, exclusions and DNS settings. Add Linux Mint and test browser, DNS, SSH where appropriate and SMB access. | Lease evidence; Windows and Linux reach the named application |
| 6: Files and printers | Create departmental shares on FILE01; configure share and NTFS permissions. Add a real authorised printer or an explicitly labelled simulated print queue. | Read/write/deny tests, spooler diagnosis and print-job evidence |
| 7: CRM | Define customer/contact/ticket workflow, then choose a maintained CRM compatible with the available resources. Keep its data and roles separate from clinical records. | Synthetic contact and ticket lifecycle, access tests and backup plan |
| 8: VPN | Introduce the firewall/router and isolated company LAN. Configure a supported VPN and scoped rules. | Host fails to reach internal apps with VPN off; succeeds with VPN on; internal DNS and RDP work through the tunnel |
| 9: Operations | Configure service monitoring, log review, backups, reboot recovery and a recovery runbook. | Stop/start alert, app recovery after reboot and restore to a test location |
| 10: Support assessment | Complete incident tickets using deliberate reversible faults. | Symptom, evidence, root cause, fix, retest and prevention recorded for each incident |

Complete and verify one phase before expanding. Record deviations from the proposed design.

## 6. Database investigation worksheet

Do not assume MedSecure uses SQL Server, MySQL, PostgreSQL or SQLite until the code and configuration establish it.

Record:
1. Engine/version and Python database driver.
2. Connection settings and where secrets are loaded; redact secret values.
3. Tables, primary keys, foreign keys and relationships.
4. How a synthetic record moves from form -> validation -> application -> database -> audit log.
5. How user/role records control access, including patient isolation where implemented.
6. A parameterised read query and a safe change using synthetic data.
7. Migration, backup and restore procedures.
8. Behaviour when the database is unavailable.

Keep the existing engine for the first working deployment. A separate DB01 is conditional: SQLite, for example, should not simply be treated as a network database server. Any engine change requires application and migration testing.

Success means restoring a usable application and database together, including required configuration and keys stored securely outside GitHub. A copied database file alone is not proof of recovery.

## 7. CRM and printing boundaries

MedSecure and CRM serve different business purposes. A CRM is not automatically part of MedSecure, and sharing their databases is not a default integration strategy. Select the CRM after defining the workflow; API integration is an optional later exercise.

A local PDF printer can demonstrate part of a print workflow, but it does not prove network-printer discovery, TCP connectivity or physical printing. Label simulated evidence accurately. Troubleshoot client queue -> print server -> spooler/driver -> printer endpoint as applicable.

## 8. Cross-platform plan and resource budget

Start with APP01 and the host. Add DC01 and one client next; introduce FILE01, CRM and the VPN router only when needed. Keep optional VMs powered off outside their exercises.

Indicative starting allocations, to revise against available RAM and measured performance:
- APP01: 2 vCPUs, 4 GB RAM, 60 GB dynamically allocated disk.
- DC01: 2 vCPUs, 2-4 GB RAM, 60 GB disk.
- Windows client: 2 vCPUs, 4 GB RAM, 64 GB disk.
- Linux desktop: 2 vCPUs, 2-4 GB RAM, 30 GB disk.
- FILE01, CRM01, DB01 and VPN router: size after product selection.

Dynamic disks still consume real storage as they grow. Reserve memory for the physical host and allow space for snapshots and backups. Use supported OS editions and legitimate installation media.

Use Linux Mint as the first additional desktop. Do not assume a macOS VM is supported on this Windows host; Oracle documents hardware restrictions. Leave macOS implementation pending a suitable supported Apple environment. A macOS support checklist can be prepared now, but mark all unperformed tests as pending.

## 9. Support incident exercises

| Ticket | Introduced fault | Required diagnosis |
|---|---|---|
| INC-001 | Client receives 169.254.x.x | Adapter state, DHCP service, scope, lease and broadcast path |
| INC-002 | App IP works but hostname fails | DNS client configuration, record and resolver reachability |
| INC-003 | Finance user can read but cannot edit | Group membership, share and NTFS permissions |
| INC-004 | MedSecure service stopped | Listener, service state, application log and safe restart |
| INC-005 | Incorrect database connection configuration | Application error, DB reachability and secret/config source |
| INC-006 | Account locks repeatedly | Lockout events, saved credentials and originating device |
| INC-007 | VPN connects but application is unreachable | Routes, return path, firewall rules and DNS |
| INC-008 | Printer queue stuck | Queue, spooler, driver and actual endpoint reachability |
| INC-009 | Departed user retains application access | AD disablement, application account, active sessions and token revocation |
| INC-010 | Server reboot interrupts service | Startup dependencies, service account, logs and recovery validation |

Change one thing at a time. Record a rollback and restore the baseline after every exercise.

## 10. GitHub deliverables

Keep this assignment within the existing MedSecure Enterprise Infrastructure Lab repository. Suggested additions:

- 10-Local-Company-Lab/ASSIGNMENT.md — this brief.
- 10-Local-Company-Lab/docs/architecture.md — implemented topology and traffic paths.
- 10-Local-Company-Lab/docs/inventory.md — actual VM and IP inventory.
- 10-Local-Company-Lab/docs/build-log.md — dated steps, reasons and results.
- 10-Local-Company-Lab/docs/database.md — actual engine and sanitised schema.
- 10-Local-Company-Lab/docs/access-matrix.md — department/resource permissions.
- 10-Local-Company-Lab/runbooks/ — deployment, onboarding, offboarding, backup and recovery.
- 10-Local-Company-Lab/tickets/ — incident reports.
- 10-Local-Company-Lab/evidence/ — sanitised screenshots and command outputs.
- 10-Local-Company-Lab/scripts/ — reviewed repeatable automation.

For every entry use: **Purpose -> Change -> Verification -> Result -> Rollback**. Keep explanations brief and useful to a reviewer.

Never commit passwords, .env files, private keys, VPN profiles containing secrets, production data, database backups or VM disks. Commit .env.example placeholders and synthetic examples instead.

Mark phases Planned, In progress or Verified. Evidence from earlier repositories is prior evidence, not proof that this new local environment works.

## 11. Assessment rubric

| Area | Weight |
|---|---|
| Architecture and resource planning | 15% |
| Networking, DNS, DHCP and remote access | 20% |
| AD, permissions and user lifecycle | 15% |
| Application and database understanding | 20% |
| CRM, printing and cross-platform support | 10% |
| Troubleshooting and recovery evidence | 15% |
| Clear, accurate GitHub documentation | 5% |

Mac-specific practical evidence is deferred until suitable hardware exists; record the limitation rather than inventing results.

## 12. First practical session

On the physical Windows host, gather:
- Installed RAM, CPU model and free disk space.
- VirtualBox version and current VM list.
- Existing virtual network modes and host-only network ranges.

Inside the existing Windows Server VM, gather:
- hostname
- ipconfig /all
- Get-WindowsFeature | Where-Object Installed
- Get-Service TermService

Share relevant output with identifiers/secrets redacted. Determine whether this is an existing DC before renaming it, assigning a new role or changing its networking.

**First milestone:** From the physical host, administer APP01 through RDP and open a working MedSecure login page in the host browser. Record its actual IP, port, service process and database engine.

## References

- Oracle VirtualBox networking: https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/networkingdetails.html
- Flask on Windows with Waitress: https://flask.palletsprojects.com/en/stable/deploying/waitress/
- Microsoft RDP administration and RDS distinction: https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/troubleshoot-remote-desktop-disconnected-errors
- Oracle guest hardware limitations: https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/BasicConcepts.html
