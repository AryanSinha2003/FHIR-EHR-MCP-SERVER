## 🚀 Project Highlights

<p align="center">
  <img src="https://img.shields.io/badge/AI%20Agent-Compatible-blueviolet?style=for-the-badge" />
  <img src="https://img.shields.io/badge/MCP%20Server-Enabled-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Deployed%20on-Render-black?style=for-the-badge&logo=render" />
  <img src="https://img.shields.io/badge/FastMCP-Framework-009688?style=for-the-badge&logo=fastapi" />
  <img src="https://img.shields.io/badge/FHIR%20Tools-20-orange?style=for-the-badge" />
</p>


# 🏥 FHIR MCP Server — Dynamic EHR Tool Server

A **Model Context Protocol (MCP)** server that exposes **20 FHIR (Fast Healthcare Interoperability Resources)** resource types as callable tools, enabling AI agents and LLMs to query a patient's complete Electronic Health Record (EHR) in real-time via a standards-compliant FHIR R4 API.

Built with **FastAPI**, transported over **Server-Sent Events (SSE)**, and designed for seamless integration with [PromptOpinion (PO)](https://promptopinion.com) workspaces and any MCP-compatible AI client.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [FHIR Resource Tools](#fhir-resource-tools)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [API Endpoints](#api-endpoints)
- [Project Structure](#project-structure)
- [Scopes](#scopes)
- [Example Usage](#example-usage)

---

## Overview

This MCP server bridges the gap between **AI/LLM agents** and **healthcare EHR systems**. Instead of requiring agents to understand raw FHIR APIs, this server wraps each FHIR resource into a simple, self-describing MCP tool that any LLM can invoke naturally in conversation.

**Key Features:**
- 🔧 **20 dedicated tools** — one per FHIR resource type extracted from real Synthea patient bundles
- 📡 **SSE transport** — real-time streaming communication between the AI client and the MCP server
- 🩺 **Full clinical coverage** — from demographics to diagnoses to billing, all queryable by an AI agent

---

## Architecture

```
┌───────────────────────┐
│   AI Agent / LLM      │
│  (e.g. Claude, GPT)   │
└──────────┬────────────┘
           │ MCP Protocol (SSE)
           ▼
┌───────────────────────┐
│   This MCP Server     │
│  (FastAPI + uvicorn)  │
│                       │
│  20 FHIR Tools        │
│  ┌─────────────────┐  │
│  │ get_patient_info │  │
│  │ get_conditions   │  │
│  │ get_observations │  │
│  │ ...18 more...    │  │
│  └─────────────────┘  │
└──────────────────────┘
```

---

## FHIR Resource Tools

The server exposes **21 tools**, organized into seven logical categories:

### 🩺 Clinical

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_info` | `Patient` | Demographics — name, DOB, gender, address, contact details |
| `get_patient_conditions` | `Condition` | Active and historical diagnoses and medical problems |
| `get_patient_observations` | `Observation` | Lab results, vital signs, and clinical measurements |
| `get_patient_diagnostic_reports` | `DiagnosticReport` | Formal reports — blood panels, imaging, pathology |
| `get_patient_procedures` | `Procedure` | Surgeries, interventions, and clinical actions performed |
| `get_patient_immunizations` | `Immunization` | Vaccination history with dates and statuses |
| `get_patient_medication_requests` | `MedicationRequest` | Active prescriptions, dosages, and ordering dates |

### 🤝 Care Coordination

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_encounters` | `Encounter` | Clinical visits — ER trips, check-ups, telehealth calls |
| `get_patient_care_plans` | `CarePlan` | Treatment strategies, goals, and follow-up actions |
| `get_patient_care_team` | `CareTeam` | People responsible for the patient's care (doctors, nurses, family) |

### 🏢 Administrative

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_practitioners` | `Practitioner` | Healthcare professionals — names and qualifications |
| `get_patient_organizations` | `Organization` | Hospitals, clinics, insurance companies |

### 💳 Billing & Insurance

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_claims` | `Claim` | Insurance claims submitted for services rendered |
| `get_patient_explanation_of_benefits` | `ExplanationOfBenefit` | Insurance response — what was paid, denied, or patient owes |

### 🔧 Devices & Supplies

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_devices` | `Device` | Medical devices — implants, prosthetics, monitors |
| `get_patient_supply_deliveries` | `SupplyDelivery` | Supply delivery records — medical supply shipments and equipment deliveries |

### 📄 Documents & Imaging

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_document_references` | `DocumentReference` | Clinical documents — discharge summaries, clinical notes, consent forms |
| `get_patient_imaging_studies` | `ImagingStudy` | Imaging studies — X-rays, MRIs, CT scans with modality and series info |

### 💊 Medication Management

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_medications` | `Medication` | Drug definitions and packaging info resolved from prescription references |
| `get_patient_medication_administrations` | `MedicationAdministration` | Actual drug administrations — dosages, routes, and timestamps |

### 🔍 Audit & Provenance

| Tool Name | FHIR Resource | Description |
|---|---|---|
| `get_patient_provenance` | `Provenance` | Audit trail — who created or modified clinical data and when |

---

## How It Works

### 1. Tool Invocation

When the AI agent calls a tool (e.g., `get_patient_conditions`), the server:

1. Parses the FHIR Bundle response
2. Formats the results into a clean, human-readable text summary
3. Returns it as an MCP `TextContent` response

---

## Prerequisites

- **Python 3.10+**
- An MCP-compatible client (e.g.Claude Desktop, or a custom MCP client)

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/sse` | SSE connection endpoint for MCP clients |
| `POST` | `/messages/` | Message handling endpoint for MCP tool calls |

### Connecting from an MCP Client

Point your MCP client to:

```
SSE Endpoint: http://localhost:8000/sse
```

---

## Scopes

The server requests the following SMART on FHIR scopes. All are set to `required: true` to ensure full clinical coverage:

```
patient/Patient.rs
patient/Condition.rs
patient/Observation.rs
patient/MedicationRequest.rs
patient/Encounter.rs
patient/Procedure.rs
patient/DiagnosticReport.rs
patient/Immunization.rs
patient/CarePlan.rs
patient/CareTeam.rs
patient/Claim.rs
patient/ExplanationOfBenefit.rs
patient/Practitioner.rs
patient/Device.rs
patient/DocumentReference.rs
patient/ImagingStudy.rs
patient/Medication.rs
patient/MedicationAdministration.rs
patient/Provenance.rs
patient/SupplyDelivery.rs
```

> **Note**: The `.rs` suffix indicates **read** access to these resources within the patient's scope.

---

## Example Usage

### From an AI Agent (Conceptual)

Once the MCP server is connected and a patient is selected in the workspace:

```
User:  "What medical conditions does this patient have?"
Agent: [calls get_patient_conditions]
       → Patient Conditions:
         - Hypertension (Status: active, Recorded: 2020-03-15)
         - Type 2 Diabetes (Status: active, Recorded: 2018-07-22)
         - Seasonal Allergic Rhinitis (Status: inactive, Recorded: 2015-04-10)
```

```
User:  "Show me their recent lab results"
Agent: [calls get_patient_observations]
       → Observations / Labs:
         - Hemoglobin A1c: 6.8 % (Date: 2024-01-12)
         - Blood Pressure Systolic: 138 mmHg (Date: 2024-01-12)
         - LDL Cholesterol: 112 mg/dL (Date: 2023-11-05)
```

```
User:  "What medications are they on?"
Agent: [calls get_patient_medication_requests]
       → Medications:
         - Metformin 500mg (Status: active, Prescribed: 2018-07-22)
         - Lisinopril 10mg (Status: active, Prescribed: 2020-03-15)
```

### Direct MCP Client Test

You can test the SSE connection with any MCP-compatible SDK:

```python
from mcp.client.sse import sse_client
from mcp import ClientSession

async with sse_client("http://localhost:8000/sse") as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        tools = await session.list_tools()
        print(f"Available tools: {[t.name for t in tools.tools]}")
```

---

## Data Source

The sample patient JSON files in this repository were generated using [**Synthea**](https://github.com/synthetichealth/synthea) — an open-source synthetic patient generator that creates realistic (but entirely fictional) FHIR patient records. These files are FHIR R4 Bundles containing all 14 resource types supported by this server.

---

<p align="center">
  Built for interoperable AI-powered healthcare
</p>
