#  Procure-to-Pay (P2P) Cycle Implementation Using SAP MM + FI

![SAP MM](https://img.shields.io/badge/SAP-MM%20Materials%20Management-0070F3?style=flat-square&logo=sap&logoColor=white)
![SAP FI](https://img.shields.io/badge/SAP-FI%20Financial%20Accounting-00A86B?style=flat-square&logo=sap&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)
![Batch](https://img.shields.io/badge/Batch-SAP%20Program%20B8-orange?style=flat-square)

> A practical walkthrough of Vendor Onboarding, Order Management, Goods Receipt, Invoice Matching, and Vendor Payment using SAP MM integrated with SAP FI.

---

##  Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Solution & Key Features](#solution--key-features)
- [Process Flow](#process-flow)
- [Detailed Process Stages](#detailed-process-stages)
- [Accounting Entries Summary](#accounting-entries-summary)
- [Automatic Account Determination (OBYC)](#automatic-account-determination-obyc)
- [Tech Stack](#tech-stack)
- [Transaction Codes Reference](#transaction-codes-reference)
- [Month-End Checklist](#month-end-checklist)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

##  Overview

This project demonstrates the end-to-end **Procure-to-Pay (P2P) cycle** implemented using **SAP Materials Management (MM)** integrated with **SAP Financial Accounting (FI)**. Every step — from creating a vendor master record to processing the final payment — is executed through SAP, with automatic financial postings generated at each stage.

The project bridges the gap between procurement operations and financial consequences, showing not just *what* each step does, but *how* it affects the general ledger in real time.

---

##  Problem Statement

Procurement without a structured system creates delays, billing disputes, and financial discrepancies that are difficult to trace. Common issues include:

- Purchase requests disappearing in email chains
- Invoices arriving that do not match what was ordered
- Late payments due to untracked approvals
- Duplicate payments and missed early-payment discounts
- Inventory figures that are unreliable at month-end

The root cause is a **coordination gap** between procurement and finance. Without a connected system, each department works from its own version of the truth — purchasing thinks a delivery is complete, finance thinks the invoice is pending, and the vendor thinks payment is overdue.

---

##  Solution & Key Features

This project implements the complete P2P cycle in **SAP MM + FI**, ensuring all procurement activities are connected and executed systematically with minimal manual intervention.

| Feature | Description |
|---|---|
|  **Single Vendor Source of Truth** | All supplier details live in one vendor master record; changes flow through every subsequent transaction automatically |
|  **Self-Running Procurement Flow** | Once a requisition is approved and converted to a PO, the system guides the rest of the process without manual handoffs |
|  **Instant Financial Reflection** | Every goods movement and invoice posting immediately creates the corresponding accounting entry in FI — no lag, no manual journal |
|  **Built-in Three-Way Match** | SAP compares the vendor's invoice against the PO price and GR quantity before any payment is allowed, protecting against overpayments |
|  **Complete Audit Trail** | Every document carries a timestamp and a reference to the preceding document, making the full procurement chain traceable in seconds |

---

##  Process Flow

```
┌─────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│  Vendor Creation │────▶│  Purchase Requisition │────▶│   Purchase Order     │
│    (XK01/MK01)  │     │       (ME51N)         │     │      (ME21N)         │
└─────────────────┘     └──────────────────────┘     └──────────┬───────────┘
                                                                  │
          ┌───────────────────────────────────────────────────────▼───────────┐
          │                                                                    │
┌─────────▼──────────┐     ┌──────────────────────┐     ┌────────────────────┐
│   Goods Receipt    │────▶│ Invoice Verification  │────▶│ Payment Processing │
│      (MIGO)        │     │       (MIRO)          │     │   (F110 / F-53)    │
└────────────────────┘     └──────────────────────┘     └────────────────────┘
```

---

##  Detailed Process Stages

### Stage 1 — Vendor Creation (`XK01 / MK01`)

The vendor master is split into three views:

- **General Data** — Vendor name, address, and bank details
- **Company Code Data** — Payment terms (Net 30) and reconciliation account (160000)
- **Purchasing Org Data** — Ordering currency and purchasing organization link

| Field | Value |
|---|---|
| Vendor Name | ABC Supplies Pvt. Ltd. |
| Company Code | TN01 |
| Payment Terms | Net 30 |
| Reconciliation Account | 160000 (Vendor Payables) |

---

### Stage 2 — Purchase Requisition (`ME51N`)

An internal document raised by a department specifying:
- What material is required
- Quantity and delivery date needed

Once approved, the purchasing team converts it to a Purchase Order using `ME21N`, maintaining a clear link between the business need and the formal commitment.

---

### Stage 3 — Purchase Order (`ME21N`)

The formal commitment sent to the vendor. It references the requisition automatically and drives all downstream steps.

| Field | Value |
|---|---|
| Document Type | NB (Standard PO) |
| Vendor | ABC Supplies Pvt. Ltd. |
| Material | Raw Material XYZ |
| Quantity | 500 Units @ USD 200.00 |
| Total Value | USD 100,000.00 |

---

### Stage 4 — Goods Receipt (`MIGO`)

When the vendor delivers goods, the warehouse confirms receipt in SAP using `MIGO` (Movement Type 101). SAP **automatically** generates the accounting entry through account determination — no manual journal required.

**Auto-Generated Accounting Entry:**
```
Dr  310000  Inventory Account       USD 100,000.00
    Cr  191100  GR/IR Clearing Account  USD 100,000.00
```

The GR/IR account acts as a **temporary liability** until the vendor invoice arrives and clears it.

---

### Stage 5 — Invoice Verification (`MIRO`)

When the vendor's invoice arrives, it is processed through `MIRO`. SAP performs a **three-way match**:

```
  Purchase Order Price  ✔
+ Goods Receipt Quantity ✔
+ Vendor Invoice Amount  ✔
= Invoice Approved for Payment
```

If values fall outside configured tolerances, the invoice is **automatically blocked** for investigation.

**Accounting Entry on Invoice Posting:**
```
Dr  191100  GR/IR Clearing Account  USD 100,000.00
    Cr  160000  Vendor Payables         USD 100,000.00
```

---

### Stage 6 — Payment Processing (`F110 / F-53`)

The automatic payment program `F110` picks up all due invoices based on payment terms and processes them in a batch run.

**Accounting Entry on Payment:**
```
Dr  160000  Vendor Payables   USD 100,000.00
    Cr  113000  Bank Account      USD 100,000.00

Status: VENDOR ACCOUNT CLEARED ✅
```

---

##  Accounting Entries Summary

| P2P Stage | Debit Account | Credit Account | Amount |
|---|---|---|---|
| **Goods Receipt** | 310000 — Inventory | 191100 — GR/IR Clearing | USD 100,000.00 |
| **Invoice Verification** | 191100 — GR/IR Clearing | 160000 — Vendor Payables | USD 100,000.00 |
| **Payment Run** | 160000 — Vendor Payables | 113000 — Bank Account | USD 100,000.00 |

>  **GR/IR Clearing Account** is a temporary holding account. Any open balance at month-end signals unmatched deliveries or uninvoiced receipts — both must be resolved before financial close.

---

##  Automatic Account Determination (OBYC)

SAP uses transaction code `OBYC` to configure the automatic link between MM movements and FI general ledger accounts. This eliminates manual account assignment entirely.

**How it works — G/L account selection is driven by:**
- **Valuation Class** associated with the material
- **Transaction Key** representing the type of posting
- **Chart of Accounts** assigned to the company code

| Transaction Key | Role in Process | Example Account |
|---|---|---|
| **BSX** | Records inventory value | 310000 — Inventory Account |
| **WRX** | Handles GR/IR clearing | 191100 — GR/IR Clearing Account |
| **GBB** | Manages offset postings | Consumption/Adjustment Account |

>  Incorrect OBYC configuration causes failed postings, wrong G/L account assignments, and financial report discrepancies.

---

##  Tech Stack

| Component | Purpose |
|---|---|
| **SAP MM** (Materials Management) | Vendor master, requisitions, purchase orders, goods receipts |
| **SAP FI** (Financial Accounting) | Vendor liability, payment processing, bank account management |
| **SAP GUI** | Desktop interface for executing all transaction codes |
| **OBYC Configuration** | Automatic account determination linking MM movements to G/L accounts |
| **ME2M** | Purchase order monitoring report |
| **MB51** | Material document list |
| **MIR5** | Invoice list report |
| **FBL1N** | Vendor line item display |

---

##  Transaction Codes Reference

| T-Code | Description |
|---|---|
| `XK01` / `MK01` | Create vendor master (central / purchasing org) |
| `ME51N` | Create purchase requisition |
| `ME21N` | Create purchase order |
| `MIGO` | Post goods receipt against purchase order |
| `MIRO` | Logistics invoice verification (3-way match) |
| `F110` | Automatic payment program (batch run) |
| `F-53` | Post outgoing payment manually |
| `FBL1N` | Vendor line item display |
| `F.13` | Automatic clearing of GR/IR open items |
| `MB51` | Material document list |
| `OBYC` | Automatic account determination configuration |

---

##  Month-End P2P Checklist

| Task | Owner | Status |
|---|---|---|
| Confirm all POs are goods-receipted | Procurement Team |  Complete |
| Clear GR/IR account open items (`F.13`) | GL Accounting |  Complete |
| Verify all invoices posted via MIRO | AP Team |  Complete |
| Execute automatic payment run (`F110`) | Treasury |  In Progress |
| Reconcile vendor subledger to GL | Finance Controller |  Pending |
| Generate vendor aging and payment report | Corporate Reporting |  Pending |

---

##  Future Improvements

- **SAP S/4HANA Migration** — Fiori interface for more intuitive approvals and monitoring, especially on mobile
- **Workflow-Driven Approvals** — Automated SAP workflows to log, track, and auto-escalate requisition approvals without email dependency
- **Advanced Analytics** — Integration with Power BI or SAP Analytics Cloud for spending pattern analysis and vendor performance dashboards
- **AI-Assisted Vendor Selection** — Machine learning on vendor performance data to proactively flag suppliers with histories of late delivery

---

## 👤 Author

**Shruti Mahapatra**
Roll Number: 2305491
Batch: SAP Program B8
Module Focus: SAP MM integrated with SAP FI

---

