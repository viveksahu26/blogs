+++
date = '2026-06-01T10:00:00+05:30'
draft = false
title = 'Understanding SBOM Merge Strategies: A Complete Guide'
categories = ['Tools', 'Best Practices']
tags = ['SBOM', 'sbomasm', 'Merge Strategies', 'CycloneDX', 'SPDX', 'SBOM Assembly', 'Hierarchical', 'Flat Merge', 'Augment Merge']
author = 'Vivek Sahu'
description = 'Master SBOM merge strategies: hierarchical, flat, assembly, and augment merge. Learn when to use each approach for combining SBOMs effectively with hands-on sbomasm examples.'
+++

![Blog header for SBOM merge strategies guide](/posts/image-merge-strategies.png)

Hey SBOM enthusiasts 👋,

If you've ever found yourself with multiple SBOMs for the same software and wondered how to combine them into a single, coherent document, you're not alone. Whether you're dealing with platform-specific builds, multi-service architectures, or scan results that need enriching, understanding merge strategies is crucial for producing accurate, usable SBOMs.

But here's the thing — choosing the wrong merge strategy can lead to unexpected results: duplicate components, broken dependency chains, or SBOMs that don't validate. Understanding when and how to use each strategy is essential for anyone working with software supply chain transparency.

In this post, we'll first explore the **conceptual merge strategies** that apply to any SBOM tool, then show you how **sbomasm** brings these strategies to life with practical, hands-on examples.

---

## Why Merge SBOMs?

Before diving into strategies, let's talk about why you'd merge SBOMs in the first place:

- **Platform-specific builds**: You have Linux and Windows builds of the same software
- **Multi-service applications**: Your product consists of several microservices, each with its own SBOM
- **Scan result enrichment**: You want to add vulnerability or license scan results to an existing SBOM
- **Vendor integrations**: Combining your internal SBOM with third-party component SBOMs
- **CI/CD artifacts**: Merging SBOMs from different build stages

Each scenario calls for a different approach, and that's where merge strategies come in.

---

## The Four Merge Strategies

There are four distinct ways to combine SBOMs, each suited for different use cases:

| Strategy | Creates New Root? | Best For |
|----------|------------------|----------|
| **Hierarchical** | ✅ Yes | Different services/modules |
| **Flat** | ✅ Yes | Same software, different platforms |
| **Assembly** | ✅ Yes | Product suites, independent tools |
| **Augment** | ❌ No | Enriching existing SBOMs |

Let's explore each one conceptually before we see how sbomasm implements them.

---

## 1. Hierarchical Merge

**Concept**: Each input SBOM becomes a self-contained sub-tree under a new root, preserving the complete component hierarchy.

### Visual Overview

```
Input SBOMs:
┌─────────────────────┐    ┌─────────────────────┐
│ SBOM A: Frontend    │    │ SBOM B: Backend     │
│ ├── frontend (root) │    │ ├── backend (root)  │
│ │   ├── react       │    │ │   ├── express     │
│ │   ├── axios       │    │ │   ├── mongoose    │
│ │   └── nginx       │    │ │   └── jwt         │
└─────────────────────┘    └─────────────────────┘

Output (Hierarchical):
Final SBOM
├── New Root (MyApp v1.0.0)
│   ├── SBOM A Root (frontend)  ← AS SUB-COMPONENT
│   │   ├── react               ← Nested under frontend
│   │   ├── axios               ← Nested under frontend
│   │   └── nginx               ← Nested under frontend
│   └── SBOM B Root (backend)   ← AS SUB-COMPONENT
│       ├── express             ← Nested under backend
│       ├── mongoose            ← Nested under backend
│       └── jwt                 ← Nested under backend
```

### When to Use

| Scenario | Why Hierarchical? |
|----------|-------------------|
| Microservices platform | Each service becomes a sub-component, maintaining service-level relationships |
| Container + Application | Base image components nested separately from app components |
| Multi-module projects | Maven/Gradle multi-module where each module is distinct |
| Different teams' components | Services from different teams that compose a platform |

### Real-World Example: Kubernetes Microservices Platform

Imagine you have an e-commerce platform running on Kubernetes with four microservices:

1. **Frontend** (React + nginx)
   - Components: react, react-router-dom, axios, nginx

2. **API Gateway** (Node.js Express)
   - Components: express, helmet, cors, jsonwebtoken

3. **Payment Service** (Go)
   - Components: stripe-go, paypal-sdk, gorilla-mux

4. **Order Service** (Java Spring Boot)
   - Components: spring-boot, hibernate, kafka-client

**Hierarchical merge creates:**

```
ecommerce-platform v1.0.0 (Application)
├── frontend v2.1.0 (Sub-component)
│   ├── react v18.2.0
│   ├── react-router-dom v6.14.0
│   ├── axios v1.4.0
│   └── nginx v1.25.0
├── api-gateway v1.5.0 (Sub-component)
│   ├── express v4.18.2
│   ├── helmet v7.0.0
│   ├── cors v2.8.5
│   └── jsonwebtoken v9.0.0
├── payment-service v3.0.0 (Sub-component)
│   ├── stripe-go v74.0.0
│   ├── paypal-sdk v1.0.0
│   └── gorilla-mux v1.8.0
└── order-service v2.2.0 (Sub-component)
    ├── spring-boot v3.1.0
    ├── hibernate v6.2.0
    └── kafka-client v3.4.0
```

**Key Characteristics**:
- ✅ Preserves complete hierarchy from each SBOM
- ✅ Each service's dependencies are nested within that service
- ✅ Best for combining SBOMs of **different** software components

---

## 2. Flat Merge

**Concept**: Everything flattened to a single level. Duplicates removed. Simplest structure.

### Visual Overview

```
Input SBOMs (Platform Variants):
┌──────────────────────────────┐    ┌──────────────────────────────┐
│ SBOM A: MyApp (Linux)        │    │ SBOM B: MyApp (Windows)      │
│ ├── MyApp v1.0.0             │    │ ├── MyApp v1.0.0             │
│ │   ├── idna v3.11           │    │ │   ├── idna v3.11           │
│ │   ├── urllib3 v2.7.0       │    │ │   ├── urllib3 v2.7.0       │
│ │   └── yarl v1.24.2         │    │ │   └── yarl v1.24.2         │
└──────────────────────────────┘    └──────────────────────────────┘

Output (Flat):
Final SBOM
├── New Root (MyApp v1.0.0)
│   ├── idna v3.11        ← FLAT (deduplicated)
│   ├── urllib3 v2.7.0      ← FLAT (deduplicated)
│   └── yarl v1.24.2        ← FLAT (deduplicated)
```

### When to Use

| Scenario | Why Flat? |
|----------|-----------|
| Platform-specific builds | Merging Linux + Windows builds of same software |
| License/Compliance Inventory | Just need a list of all components for legal review |
| Simple BOM | Quick inventory without relationship complexity |
| Large-scale aggregation | When you don't care about which component came from where |

### Real-World Example: Multi-Platform Python Application

You have the same Python application built for Linux and Windows:

- **Linux SBOM**: cs.template v2026.2.3.0 with Python libs (idna, multidict, yarl)
- **Windows SBOM**: cs.template v2026.2.3.0 with same Python libs

**Flat merge creates:**

```
cs.template v2026.2.3.0 (Application)
├── idna v3.11        ← From both SBOMs, deduplicated
├── multidict v6.7.1  ← From both SBOMs, deduplicated
├── propcache v0.5.2  ← From both SBOMs, deduplicated
├── urllib3 v2.7.0    ← From both SBOMs, deduplicated
└── yarl v1.24.2      ← From both SBOMs, deduplicated
```

**Key Characteristics**:
- ✅ Simplest possible structure
- ✅ Deduplicates components automatically
- ✅ All components at the same level
- ✅ Best for merging SBOMs of the **same** software for different platforms

---

## 3. Assembly Merge

**Concept**: Primary components become sub-components of the new root, but all other components stay at the top level. A middle ground between hierarchical and flat.

### Visual Overview

```
Input SBOMs:
┌─────────────────────┐    ┌─────────────────────┐
│ SBOM A: Tool A      │    │ SBOM B: Tool B      │
│ ├── tool-a (root)   │    │ ├── tool-b (root)   │
│ │   ├── dep1        │    │ │   ├── dep3        │
│ │   └── dep2        │    │ │   └── dep4        │
└─────────────────────┘    └─────────────────────┘

Output (Assembly):
Final SBOM
├── New Root (Tool Suite v1.0.0)
│   ├── tool-a (Sub-component)  ← Primary as sub-component
│   └── tool-b (Sub-component)  ← Primary as sub-component
├── dep1      ← TOP LEVEL
├── dep2      ← TOP LEVEL
├── dep3      ← TOP LEVEL
└── dep4      ← TOP LEVEL
```

### When to Use

| Scenario | Why Assembly? |
|----------|---------------|
| Product Suite | Combining independent products that share some dependencies |
| Library Collection | Creating a bundle of related libraries |
| Plugin System | Main app + plugins where plugins are first-class |
| Distribution Package | OS package that bundles multiple independent tools |

### Real-World Example: Security Toolkit

You want to create an SBOM for a security toolkit containing:
- cosign (container signing)
- syft (SBOM generation)
- grype (vulnerability scanning)
- trivy (container scanning)

**Assembly merge creates:**

```
security-toolkit v1.0.0 (Application)
├── cosign v2.0.0 (Sub-component)
├── syft v0.80.0 (Sub-component)
├── grype v0.70.0 (Sub-component)
├── trivy v0.45.0 (Sub-component)
├── sigstore v1.0.0        ← Top level
├── go-containerregistry ← Top level
├── anchore-db            ← Top level
└── fanal                 ← Top level
```

**Key Characteristics**:
- ✅ Primary components are nested under the new root
- ✅ All other components are flat (top-level)
- ✅ Good balance between hierarchy and flatness
- ✅ Best for bundling **related but independent** products

---

## 4. Augment Merge

**Concept**: Enriches the primary SBOM with data from secondary SBOMs without creating a new root. This is fundamentally different from the other strategies.

### Visual Overview

```
Other Merges:  NEW ROOT ← SBOM A + SBOM B + SBOM C
Augment Merge: PRIMARY ← PRIMARY + (SBOM B data) + (SBOM C data)
```

**Think**: "Update/Enrich" not "Combine/Assemble"

### How It Works

1. **Component Matching**: Components are matched between primary and secondary SBOMs using:
   - Name
   - Version
   - PURL
   - CPE

2. **Two Modes**:
   - `if-missing-or-empty` (default): Only fills fields that are empty/missing in primary
   - `overwrite`: Replaces primary fields with secondary values

3. **What Gets Merged**:
   - Description, Author, Publisher, Group
   - Scope, Copyright
   - PackageURL, CPE, SWID
   - Supplier, Licenses, Hashes
   - ExternalReferences, Properties

### When to Use

| Scenario | Example |
|----------|---------|
| Add vulnerability scan results | Enrich base SBOM with Trivy/Grype scan results |
| Add license scan data | Enrich with FOSSology or ScanCode license data |
| Merge vendor SBOM | Combine internal SBOM with vendor-provided SBOM |
| Progressive enhancement | Multiple passes with different data sources |

### Real-World Example: Enriching with Vulnerability Scan

You have a base SBOM and want to add vulnerability data from a security scan:

```
Input:
- base-sbom.cdx.json (your application SBOM)
- vuln-scan.cdx.json (scan results with CVEs)

Augment Merge Output:
base-sbom.cdx.json (ENRICHED)
├── Same components as before
├── Same dependencies as before
└── NEW: vulnerabilities section with CVE data
```

**Key Characteristics**:
- ❌ Does NOT create a new root
- ✅ Preserves the primary SBOM's identity and version
- ✅ Merges matching components, adds non-matching ones
- ✅ Best for **enriching** existing SBOMs with additional data

---

## Decision Matrix

| Use Case | Hierarchical | Assembly | Flat | Augment |
|----------|-------------|----------|------|---------|
| Microservices platform | ✅ | ⚠️ | ❌ | ❌ |
| Container + App layers | ✅ | ⚠️ | ❌ | ❌ |
| Product suite (independent) | ❌ | ✅ | ⚠️ | ❌ |
| Same software, multi-platform | ❌ | ❌ | ✅ | ✅ |
| License inventory only | ❌ | ❌ | ✅ | ❌ |
| Enrich with scan data | ❌ | ❌ | ❌ | ✅ |
| CI/CD pipeline artifacts | ⚠️ | ⚠️ | ✅ | ❌ |

**Legend**: ✅ Best | ⚠️ Works | ❌ Wrong choice

---

## sbomasm: Bringing Strategies to Life

Now that we understand the conceptual merge strategies, let's see how **sbomasm** implements them.

**sbomasm** is a command-line tool from Interlynk that makes SBOM assembly, editing, and enrichment straightforward. It supports both CycloneDX and SPDX formats, and implements all four merge strategies we've discussed.

### Strategy Flags in sbomasm

| Strategy | sbomasm Flag | Creates New Root? |
|----------|-------------|------------------|
| Hierarchical | `-m` (default) | ✅ Yes |
| Flat | `-f` | ✅ Yes |
| Assembly | `-a` | ✅ Yes |
| Augment | `--augmentMerge` | ❌ No |

---

## Hands-On with sbomasm

### Hierarchical Merge: Kubernetes Microservices

Remember our e-commerce platform with four microservices? Here's how to create a unified SBOM:

```bash
sbomasm assemble \
  -n "ecommerce-platform" \
  -v "1.0.0" \
  -t application \
  frontend-svc.cdx.json \
  api-gateway.cdx.json \
  payment-svc.cdx.json \
  order-svc.cdx.json \
  -o ecommerce-platform.cdx.json
```

**Result**: Each service becomes a sub-component with its dependencies nested underneath.

### Flat Merge: Multi-Platform Python Application

Merging platform-specific builds of the same Python application:

```bash
sbomasm assemble --flat-merge \
  -n "cs.template" \
  -v "2026.2.3.0" \
  -t application \
  cs.template.linux.python_sbom.cdx.json \
  cs.template.win32.python_sbom.cdx.json \
  -o cs.template.unified.cdx.json
```

**Result**: Single SBOM with deduplicated Python libraries (idna, multidict, yarl, etc.).

### Assembly Merge: Security Toolkit

Creating a security toolkit SBOM:

```bash
sbomasm assemble --assembly-merge \
  -n "security-toolkit" \
  -v "1.0.0" \
  -t application \
  cosign.json \
  syft.json \
  grype.json \
  trivy.json \
  -o security-toolkit.cdx.json
```

**Result**: Each tool is a sub-component; shared dependencies are at top level.

### Augment Merge: Adding Scan Results

Enriching your base SBOM with vulnerability scan data:

```bash
sbomasm assemble --augmentMerge \
  --primary base-sbom.cdx.json \
  vuln-scan-results.cdx.json \
  -o enriched-sbom.cdx.json
```

**Note**: No `-n/-v/-t` flags needed — the primary SBOM's identity is preserved.

---

## Common Pitfalls and How to Avoid Them

### Pitfall 1: Using Hierarchical Merge for Platform Builds

**Wrong**:
```bash
sbomasm assemble \
  linux.sbom.json windows.sbom.json \
  -n "myapp" -v "1.0.0"
```

**Problem**: Creates nested structure for identical SBOMs, causing empty components and duplicate dependencies.

**Right**:
```bash
sbomasm assemble --flat-merge \
  linux.sbom.json windows.sbom.json \
  -n "myapp" -v "1.0.0"
```

### Pitfall 2: Forgetting `--primary` with Augment Merge

**Wrong**:
```bash
sbomasm assemble --augmentMerge sbom1.json sbom2.json
```

**Problem**: Augment merge requires specifying which SBOM is the primary one.

**Right**:
```bash
sbomasm assemble --augmentMerge \
  --primary sbom1.json \
  sbom2.json
```

### Pitfall 3: Expecting New Root with Augment Merge

**Wrong Understanding**: Augment merge creates a new parent SBOM.

**Correct Understanding**: Augment merge keeps your primary SBOM intact and enriches it. No new root is created.

---

## Summary

| Strategy | Creates New Root? | Use When | sbomasm Flag |
|----------|------------------|----------|--------------|
| **Hierarchical** | Yes | Merging SBOMs of **different** components (microservices) | `-m` |
| **Flat** | Yes | Merging SBOMs of the **same** software (platform variants) | `-f` |
| **Assembly** | Yes | Bundling **related but independent** products | `-a` |
| **Augment** | No | **Enriching** an existing SBOM with scan/vendor data | `--augmentMerge` |

## Resources

- sbomasm GitHub: <https://github.com/interlynk-io/sbomasm>
- Documentation: <https://github.com/interlynk-io/sbomasm/tree/main/docs>
- CycloneDX Spec: <https://cyclonedx.org/specification/overview/>
- SPDX Spec: <https://spdx.dev/specifications/>

If you loved this project, show the love back by starring ⭐ the [repo](https://github.com/interlynk-io/sbomasm).

Have questions or need help with a specific use case? File an [issue](https://github.com/interlynk-io/sbomasm/issues/new) — we'd love to help!

