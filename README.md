# 🛡️ devops-policies-cnspec-vmware

[![VMware vSphere](https://img.shields.io/badge/VMware-vSphere_9.0-607078?style=flat-square&logo=vmware&logoColor=white)](https://www.vmware.com/products/cloud-infrastructure/vsphere)
[![Mondoo](https://img.shields.io/badge/Mondoo-cnspec_|_cnquery-4C35E0?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjAiIGhlaWdodD0iMjAiIHZpZXdCb3g9IjAgMCAyMCAyMCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48Y2lyY2xlIGN4PSIxMCIgY3k9IjEwIiByPSI4IiBmaWxsPSJ3aGl0ZSIvPjwvc3ZnPg==&logoColor=white)](https://mondoo.com/cnspec)
[![Buildkite](https://img.shields.io/badge/Buildkite-Pipelines-14CC80?style=flat-square&logo=buildkite&logoColor=white)](https://buildkite.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)
[![Policy as Code](https://img.shields.io/badge/Policy_as_Code-MQL-blue?style=flat-square)](#writing-custom-policies)

> Security scanning, asset inventory, and compliance-as-code for VMware vSphere Foundation 9.0 using **cnspec** and **cnquery** by Mondoo.

---

## 📋 What This Repo Does

| Tool | Purpose |
|---|---|
| 🔒 **cnspec** | Security policy scanning, vulnerability assessment, compliance checks |
| 🔍 **cnquery** | Asset inventory, health queries, capacity reporting, incident response |

---

## 🗂️ Repository Structure

```
├── 📂 .buildkite/
│   ├── 📄 pipeline.yml              # Main Buildkite pipeline
│   ├── 📄 pipeline.scheduled.yml    # Scheduled scans (daily/weekly)
│   └── 📂 scripts/                  # Pipeline helper scripts
├── 📂 policies/
│   ├── 🔒 security/                 # ESXi & vCenter hardening policies
│   ├── 📋 compliance/               # CIS, STIG, custom frameworks
│   └── ⚙️ operational/              # Operational best-practice policies
├── 📂 queries/
│   ├── 🖥️ inventory/                # Asset inventory query packs
│   ├── 🩺 health/                   # Health check query packs
│   └── 📊 capacity/                 # Capacity & utilization query packs
├── 📂 scripts/                      # Utility scripts
└── 📂 reports/                      # Report templates & output (gitignored)
```

---

## ✅ Prerequisites

| Requirement | Version |
|---|---|
| ![cnspec](https://img.shields.io/badge/cnspec-≥_11.x-4C35E0?style=flat-square) | Security & compliance scanner |
| ![cnquery](https://img.shields.io/badge/cnquery-≥_11.x-4C35E0?style=flat-square) | Asset inventory & query engine |
| ![Buildkite](https://img.shields.io/badge/Buildkite-Agent-14CC80?style=flat-square&logo=buildkite&logoColor=white) | With network access to vCenter |
| ![Mondoo](https://img.shields.io/badge/Mondoo-Platform-4C35E0?style=flat-square) | Optional — for dashboards |

---

## 🚀 Quick Start

```bash
# Install cnspec + cnquery
bash -c "$(curl -sSL https://install.mondoo.com/sh)"

# Interactive shell — explore your vSphere environment
cnquery shell vsphere administrator@vsphere.local@vcenter.lab.local --ask-pass

# Run a security scan with custom policies
cnspec scan vsphere administrator@vsphere.local@vcenter.lab.local \
  --ask-pass \
  --policy-bundle policies/security/esxi-hardening.mql.yaml

# Run an inventory query pack
cnquery scan vsphere administrator@vsphere.local@vcenter.lab.local \
  --ask-pass \
  --querypack-bundle queries/inventory/full-inventory.mql.yaml
```

---

## 🏗️ Buildkite Pipelines

| Pipeline | Trigger | Description |
|---|---|---|
| ⚡ `pipeline.yml` | PR / push | Validates policies, lint, dry-run |
| 🕐 `pipeline.scheduled.yml` | Cron (daily) | Full security scan + inventory + vulnerability assessment |

### 🔑 Required Buildkite Secrets

| Secret | Description |
|---|---|
| `VSPHERE_SERVER` | vCenter FQDN |
| `VSPHERE_USER` | SSO username (`user@domain`) |
| `VSPHERE_PASSWORD` | SSO password |
| `MONDOO_SERVICE_TOKEN` | Mondoo Platform token (optional) |
| `GITGUARDIAN_API_KEY` | GitGuardian secret scanning |
| `SLACK_WEBHOOK_URL` | Slack notifications (optional) |

---

## 📦 Policies & Query Packs

### 🔒 Security Policies (cnspec)

| Policy | Checks | Target |
|---|---|---|
| [`esxi-hardening.mql.yaml`](policies/security/esxi-hardening.mql.yaml) | 16 | ESXi hosts — SSH, lockdown, TLS 1.3, FIPS, firewall, vSwitch |
| [`vcenter-hardening.mql.yaml`](policies/security/vcenter-hardening.mql.yaml) | 10 | vCenter — password policy, session timeout, TLS, logging |
| [`vm-hardening.mql.yaml`](policies/security/vm-hardening.mql.yaml) | 13 | VMs — isolation, devices, HW v22, EFI, Tools |

### 📋 Compliance Policies (cnspec)

| Policy | Framework | Controls |
|---|---|---|
| [`cis-esxi9.mql.yaml`](policies/compliance/cis-esxi9.mql.yaml) | CIS Benchmark | 16 scored controls across 7 sections |

### ⚙️ Operational Policies (cnspec)

| Policy | Checks | Focus |
|---|---|---|
| [`best-practices.mql.yaml`](policies/operational/best-practices.mql.yaml) | 9 | HA, DRS, admission control, snapshots, host state |

### 🔍 Query Packs (cnquery)

| Pack | Queries | Purpose |
|---|---|---|
| [`full-inventory.mql.yaml`](queries/inventory/full-inventory.mql.yaml) | 8 | Complete asset inventory — DCs, clusters, hosts, VMs, datastores |
| [`host-health.mql.yaml`](queries/health/host-health.mql.yaml) | 10 | Uptime, services, hardware sensors, alarms, snapshots |
| [`cluster-capacity.mql.yaml`](queries/capacity/cluster-capacity.mql.yaml) | 8 | Utilization, overcommit, powered-off waste, storage |

---

## ✏️ Writing Custom Policies

Policies use MQL (Mondoo Query Language). Example:

```yaml
policies:
  - uid: my-esxi-check
    name: "My ESXi Check"
    groups:
      - checks:
          - uid: ssh-disabled

queries:
  - uid: ssh-disabled
    title: "SSH must be disabled on ESXi"
    mql: |
      esxi.host {
        services.where(key == "TSM-SSH") {
          running == false
        }
      }
```

📖 See [MQL vSphere Resource Reference](https://mondoo.com/docs/mql/resources/vsphere-pack/) for all available resources.

---

## 🧪 Local Testing

```bash
# Copy and configure credentials
cp .env.example .env

# Run everything
./scripts/run-local.sh all

# Or pick a specific mode
./scripts/run-local.sh scan       # Security scans only
./scripts/run-local.sh vuln       # Vulnerability scan only
./scripts/run-local.sh inventory  # Asset inventory
./scripts/run-local.sh health     # Host health checks
./scripts/run-local.sh capacity   # Capacity report
./scripts/run-local.sh shell      # Interactive cnquery shell
```

---

## 📄 License

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)
