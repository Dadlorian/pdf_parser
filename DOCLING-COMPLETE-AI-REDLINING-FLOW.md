# Complete AI-Driven Contract Redlining System with IBM Docling

**Date:** November 13, 2025
**Purpose:** End-to-end architecture for AI-powered contract redlining with universal document support
**Status:** Complete System Design

---

## Executive Summary

This document describes a **complete AI-driven contract redlining system** that:

1. **Accepts any document format** (PDF, DOCX, PPTX, RTF, etc.)
2. **Parses document structure** using IBM Docling (paragraphs, sections, tables, bullets)
3. **Sends each chunk to multiple AI agents** for contract analysis
4. **AI agents determine redlining recommendations** (strike-through, highlight, comment, accept)
5. **Visualizes redlining on the actual PDF** with real-time highlighting
6. **Lawyers review and accept/reject AI recommendations**

### The Key Insight

**Docling creates a "map" of the document structure with precise coordinates. This map enables:**
- **AI Analysis:** Each structural element (paragraph, table, bullet) sent to AI agents as a discrete chunk
- **Visual Rendering:** Bounding boxes from Docling overlay precisely on the original PDF
- **Interactive Redlining:** Click a highlighted section to accept/reject AI recommendations

**The PDF stays intact. Docling just tells you WHERE everything is and WHAT it is.**

---

## Table of Contents

1. [Complete System Architecture](#complete-system-architecture)
2. [Contract Redlining Context](#contract-redlining-context)
3. [End-to-End Data Flow](#end-to-end-data-flow)
4. [Docling Integration Details](#docling-integration-details)
5. [AI Agent Integration](#ai-agent-integration)
6. [Visual Redlining Rendering](#visual-redlining-rendering)
7. [Complete Code Implementation](#complete-code-implementation)
8. [Universal Document Support](#universal-document-support)
9. [Deployment Architecture](#deployment-architecture)
10. [Complete Example Walkthrough](#complete-example-walkthrough)

---

## Complete System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPLETE AI REDLINING SYSTEM                     │
└─────────────────────────────────────────────────────────────────────┘

                         USER UPLOADS CONTRACT
                                   ↓
┌─────────────────────────────────────────────────────────────────────┐
│ STAGE 1: DOCUMENT INGESTION & CONVERSION                           │
│                                                                     │
│  Input: contract.docx, contract.pdf, contract.pptx, etc.          │
│    ↓                                                                │
│  Format Detection → If not PDF, convert to PDF                     │
│    ↓                                                                │
│  Output: Normalized PDF + Original file                            │
└─────────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────────┐
│ STAGE 2: DOCLING STRUCTURE EXTRACTION                              │
│                                                                     │
│  Docling Parser (Python):                                          │
│    ├─ Extract text tokens + geometric coordinates                  │
│    ├─ AI layout analysis (computer vision models)                  │
│    ├─ Identify elements:                                           │
│    │   • Section headers ("1. Payment Terms")                      │
│    │   • Paragraphs (contract clauses)                             │
│    │   • Tables (pricing, deliverables)                            │
│    │   • Bullet lists (terms, conditions)                          │
│    │   • Signatures, dates, etc.                                   │
│    └─ Determine reading order (P1, P2, P3...)                      │
│                                                                     │
│  Output: Structured JSON with bounding boxes                       │
│  {                                                                  │
│    sections: [                                                      │
│      {                                                              │
│        id: "p1",                                                    │
│        type: "section-header",                                      │
│        text: "1. Payment Terms",                                    │
│        bbox: { x: 72, y: 100, width: 200, height: 24 },           │
│        pageNumber: 1                                                │
│      },                                                             │
│      {                                                              │
│        id: "p2",                                                    │
│        type: "paragraph",                                           │
│        text: "Client shall pay $100,000 within 30 days...",        │
│        bbox: { x: 72, y: 140, width: 468, height: 60 },           │
│        pageNumber: 1                                                │
│      },                                                             │
│      ...                                                            │
│    ]                                                                │
│  }                                                                  │
└─────────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────────┐
│ STAGE 3: AI AGENT ANALYSIS (PARALLEL)                              │
│                                                                     │
│  For each section (P1, P2, P3...):                                 │
│    ↓                                                                │
│  ┌────────────────────────────────────────────────────────┐        │
│  │ Send to Multiple AI Agents in Parallel:               │        │
│  │                                                        │        │
│  │  Agent 1: Compliance Review                           │        │
│  │    → Check for regulatory compliance                  │        │
│  │    → Flag risky clauses                               │        │
│  │                                                        │        │
│  │  Agent 2: Legal Risk Assessment                       │        │
│  │    → Identify liability issues                        │        │
│  │    → Check payment terms                              │        │
│  │                                                        │        │
│  │  Agent 3: Fairness Analysis                           │        │
│  │    → Check for one-sided terms                        │        │
│  │    → Identify negotiation opportunities               │        │
│  │                                                        │        │
│  │  Agent 4: Standard Clause Comparison                  │        │
│  │    → Compare to company standard clauses              │        │
│  │    → Detect deviations                                │        │
│  └────────────────────────────────────────────────────────┘        │
│                                                                     │
│  AI Agent Input (per section):                                     │
│  {                                                                  │
│    sectionId: "p2",                                                 │
│    type: "paragraph",                                               │
│    text: "Client shall pay $100,000 within 30 days...",           │
│    context: {                                                       │
│      documentType: "service-agreement",                            │
│      previousSection: "1. Payment Terms",                          │
│      nextSection: "..."                                             │
│    }                                                                │
│  }                                                                  │
│                                                                     │
│  AI Agent Output (per section):                                    │
│  {                                                                  │
│    sectionId: "p2",                                                 │
│    recommendations: [                                               │
│      {                                                              │
│        agent: "Legal Risk Assessment",                             │
│        action: "highlight",                                        │
│        severity: "high",                                            │
│        reason: "Payment terms favor client heavily",               │
│        suggestion: "Add late payment penalty clause",              │
│        confidence: 0.92                                             │
│      },                                                             │
│      {                                                              │
│        agent: "Fairness Analysis",                                 │
│        action: "strike-through",                                   │
│        severity: "medium",                                          │
│        reason: "30 days is too long for our standard terms",       │
│        suggestion: "Change to 15 days",                            │
│        confidence: 0.85                                             │
│      }                                                              │
│    ]                                                                │
│  }                                                                  │
│                                                                     │
│  Output: Aggregated AI recommendations for all sections            │
└─────────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────────┐
│ STAGE 4: REDLINING VISUALIZATION (FRONTEND)                        │
│                                                                     │
│  1. Render original PDF with PDF.js                                │
│     └─ PDF stays intact, pixel-perfect rendering                   │
│                                                                     │
│  2. Overlay Docling bounding boxes on PDF canvas                   │
│     └─ For each section, draw box at bbox coordinates              │
│                                                                     │
│  3. Apply AI recommendation styling                                │
│     ├─ Highlight (yellow) = AI suggests review                     │
│     ├─ Strike-through (red) = AI suggests deletion                 │
│     ├─ Underline (green) = AI suggests addition                    │
│     └─ Comment icon (blue) = AI has suggestions                    │
│                                                                     │
│  4. Interactive controls                                            │
│     ├─ Click section → View AI analysis                            │
│     ├─ Accept recommendation → Apply redlining                     │
│     ├─ Reject recommendation → Clear styling                       │
│     └─ Modify → Lawyer edits suggestion                            │
│                                                                     │
│  Visual Example:                                                    │
│  ┌────────────────────────────────────────────┐                    │
│  │  Contract PDF Viewer                       │                    │
│  │                                             │                    │
│  │  [P1] 1. Payment Terms                     │ ← Header (blue)    │
│  │                                             │                    │
│  │  [P2] Client shall pay $100,000 within     │ ← Highlighted      │
│  │       30 days of invoice date...           │   (yellow + red)   │
│  │       ↑ AI: Change to 15 days ↑            │   AI tooltip       │
│  │                                             │                    │
│  │  [P3] • Deliverable 1: Website design      │ ← Bullet (green)   │
│  │       • Deliverable 2: Mobile app          │                    │
│  │                                             │                    │
│  │  [P4] ┌─────────┬──────────┬───────┐       │ ← Table (normal)   │
│  │       │ Item    │ Quantity │ Price │       │                    │
│  │       ├─────────┼──────────┼───────┤       │                    │
│  │       │ Design  │ 1        │ $50k  │       │                    │
│  │       └─────────┴──────────┴───────┘       │                    │
│  │                                             │                    │
│  │  Sidebar: AI Analysis for P2               │                    │
│  │  ┌──────────────────────────────────────┐  │                    │
│  │  │ Legal Risk (High):                   │  │                    │
│  │  │ Payment terms favor client heavily.  │  │                    │
│  │  │ Suggest: Add late payment penalty    │  │                    │
│  │  │                                       │  │                    │
│  │  │ [Accept] [Reject] [Modify]           │  │                    │
│  │  └──────────────────────────────────────┘  │                    │
│  └────────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────────┐
│ STAGE 5: LAWYER REVIEW & FINALIZATION                              │
│                                                                     │
│  Lawyer reviews AI recommendations:                                 │
│    ├─ Accepts some suggestions                                     │
│    ├─ Rejects others                                               │
│    ├─ Modifies some                                                │
│    └─ Adds manual redlines                                         │
│                                                                     │
│  Output: Final redlined contract with tracked changes              │
│          Export as PDF with annotations, Word track changes, etc.  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Contract Redlining Context

### What is Contract Redlining?

In contract law, **redlining** is the process of marking up a contract draft to show proposed changes:

1. **Strike-through** (deletion): Remove unfavorable clauses
   - Example: ~~Client may terminate without penalty~~

2. **Highlight** (review needed): Flag clauses requiring attention
   - Example: <mark>Payment due within 30 days</mark>

3. **Underline** (addition): Propose new language
   - Example: <u>Late payments subject to 5% penalty</u>

4. **Comment** (note): Add explanations or questions
   - Example: 💬 "This conflicts with section 7.2"

### Why Structural Granularity Matters

Lawyers redline at the **structural level**, not word-level:
- ❌ Not: "Delete words 47-52 on page 3"
- ✅ Instead: "Delete paragraph P7 (liability cap clause)"

**This is why Docling is perfect:**
- Docling identifies structural elements (paragraphs, sections, tables)
- Each element gets a unique ID (P1, P2, P3...)
- Lawyers can say "Flag P7 for deletion" - everyone knows what that means

---

## End-to-End Data Flow

### Flow Diagram with Data Structures

```
USER UPLOADS CONTRACT
  ↓
┌─────────────────────────────────────┐
│ INPUT: contract.docx                │
│ {                                   │
│   filename: "service-agreement.docx"│
│   size: 2.3 MB                      │
│   uploadedBy: "lawyer@firm.com"     │
│ }                                   │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────┐
│ CONVERSION: DOCX → PDF              │
│ (LibreOffice)                       │
│                                     │
│ Output: service-agreement.pdf       │
└─────────────────────────────────────┘
  ↓
┌─────────────────────────────────────────────────────────────┐
│ DOCLING PARSING                                             │
│                                                             │
│ Input: service-agreement.pdf                               │
│ Output: Structured JSON                                    │
│ {                                                           │
│   documentId: "doc_12345",                                 │
│   filename: "service-agreement.pdf",                       │
│   totalPages: 8,                                            │
│   sections: [                                               │
│     {                                                       │
│       id: "p1",                                             │
│       readingOrder: 1,                                      │
│       type: "section-header",                               │
│       text: "1. PAYMENT TERMS",                            │
│       bbox: { x: 72, y: 100, w: 200, h: 24, page: 1 },    │
│       wordCount: 2                                          │
│     },                                                      │
│     {                                                       │
│       id: "p2",                                             │
│       readingOrder: 2,                                      │
│       type: "paragraph",                                    │
│       text: "Client shall pay Contractor the sum of...",   │
│       bbox: { x: 72, y: 140, w: 468, h: 60, page: 1 },    │
│       wordCount: 47                                         │
│     },                                                      │
│     {                                                       │
│       id: "p3",                                             │
│       readingOrder: 3,                                      │
│       type: "bullet-list",                                  │
│       items: [                                              │
│         "• Phase 1: Requirements gathering",               │
│         "• Phase 2: Design and development",               │
│         "• Phase 3: Testing and deployment"                │
│       ],                                                    │
│       text: "• Phase 1: Requirements... [full text]",      │
│       bbox: { x: 90, y: 220, w: 450, h: 75, page: 1 },    │
│       wordCount: 24                                         │
│     },                                                      │
│     {                                                       │
│       id: "p4",                                             │
│       readingOrder: 4,                                      │
│       type: "table",                                        │
│       structure: {                                          │
│         rows: 4,                                            │
│         cols: 3,                                            │
│         data: [                                             │
│           ["Deliverable", "Timeline", "Cost"],             │
│           ["Website", "3 months", "$50,000"],              │
│           ["Mobile App", "4 months", "$75,000"],           │
│           ["Total", "-", "$125,000"]                       │
│         ]                                                   │
│       },                                                    │
│       bbox: { x: 72, y: 320, w: 468, h: 120, page: 1 },   │
│       wordCount: 18                                         │
│     },                                                      │
│     ... (more sections)                                     │
│   ],                                                        │
│   metadata: {                                               │
│     parsedAt: "2025-11-13T18:00:00Z",                      │
│     doclingVersion: "1.0.0",                               │
│     confidence: 0.96                                        │
│   }                                                         │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
  ↓
┌─────────────────────────────────────────────────────────────┐
│ AI AGENT ANALYSIS (Multiple Agents, Parallel)              │
│                                                             │
│ For section P2:                                             │
│ Input to AI Agents:                                         │
│ {                                                           │
│   sectionId: "p2",                                          │
│   type: "paragraph",                                        │
│   text: "Client shall pay Contractor the sum of...",       │
│   documentContext: {                                        │
│     documentType: "service-agreement",                      │
│     parties: ["Client Corp", "Contractor LLC"],            │
│     jurisdiction: "New York",                               │
│     previousSection: { id: "p1", text: "1. PAYMENT..." },  │
│     nextSection: { id: "p3", text: "• Phase 1..." }        │
│   }                                                         │
│ }                                                           │
│                                                             │
│ Agent 1: Compliance Review                                 │
│ → Analyzes: Regulatory compliance, legal requirements      │
│ → Output:                                                   │
│ {                                                           │
│   agent: "compliance-reviewer",                            │
│   sectionId: "p2",                                          │
│   findings: [                                               │
│     {                                                       │
│       severity: "low",                                      │
│       action: "accept",                                     │
│       reason: "Payment terms comply with NY law",          │
│       confidence: 0.94                                      │
│     }                                                       │
│   ]                                                         │
│ }                                                           │
│                                                             │
│ Agent 2: Legal Risk Assessment                             │
│ → Analyzes: Liability, payment terms, termination clauses  │
│ → Output:                                                   │
│ {                                                           │
│   agent: "risk-assessor",                                  │
│   sectionId: "p2",                                          │
│   findings: [                                               │
│     {                                                       │
│       severity: "high",                                     │
│       action: "highlight",                                  │
│       reason: "Payment amount unusually high for scope",   │
│       suggestion: "Negotiate to $100,000 total",           │
│       confidence: 0.87                                      │
│     },                                                      │
│     {                                                       │
│       severity: "medium",                                   │
│       action: "comment",                                    │
│       reason: "No late payment penalty specified",         │
│       suggestion: "Add: 'Late payments subject to 5%...'", │
│       confidence: 0.91                                      │
│     }                                                       │
│   ]                                                         │
│ }                                                           │
│                                                             │
│ Agent 3: Fairness Analyzer                                 │
│ → Analyzes: Balanced terms, one-sided clauses              │
│ → Output:                                                   │
│ {                                                           │
│   agent: "fairness-analyzer",                              │
│   sectionId: "p2",                                          │
│   findings: [                                               │
│     {                                                       │
│       severity: "medium",                                   │
│       action: "strike-through",                            │
│       reason: "Payment schedule favors client heavily",    │
│       suggestion: "Add milestone-based payments",          │
│       confidence: 0.82                                      │
│     }                                                       │
│   ]                                                         │
│ }                                                           │
│                                                             │
│ AGGREGATED OUTPUT for P2:                                  │
│ {                                                           │
│   sectionId: "p2",                                          │
│   redlineRecommendations: [                                │
│     {                                                       │
│       action: "highlight",                                  │
│       color: "yellow",                                      │
│       severity: "high",                                     │
│       agents: ["risk-assessor"],                           │
│       reason: "Payment amount unusually high",             │
│       suggestion: "Negotiate to $100,000 total"            │
│     },                                                      │
│     {                                                       │
│       action: "strike-through",                            │
│       color: "red",                                         │
│       severity: "medium",                                   │
│       agents: ["fairness-analyzer"],                       │
│       reason: "Payment schedule favors client",            │
│       suggestion: "Add milestone-based payments"           │
│     },                                                      │
│     {                                                       │
│       action: "comment",                                    │
│       color: "blue",                                        │
│       severity: "medium",                                   │
│       agents: ["risk-assessor"],                           │
│       reason: "No late payment penalty",                   │
│       suggestion: "Add late payment clause"                │
│     }                                                       │
│   ],                                                        │
│   overallRisk: "high",                                      │
│   requiresLawyerReview: true                               │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
  ↓
┌─────────────────────────────────────────────────────────────┐
│ FRONTEND RENDERING                                          │
│                                                             │
│ Data sent to frontend:                                      │
│ {                                                           │
│   pdfUrl: "/pdfs/service-agreement.pdf",                   │
│   sections: [ ... all sections from Docling ... ],         │
│   redlineRecommendations: {                                │
│     "p2": [ ... AI recommendations ... ],                  │
│     "p5": [ ... AI recommendations ... ],                  │
│     "p7": [ ... AI recommendations ... ],                  │
│     ...                                                     │
│   }                                                         │
│ }                                                           │
│                                                             │
│ Frontend renders:                                           │
│ 1. PDF.js renders original PDF                             │
│ 2. Overlay canvas for redline visualization                │
│ 3. For each section with recommendations:                  │
│    - Draw bounding box at section.bbox coordinates         │
│    - Apply styling based on recommendation.action          │
│    - Add tooltip with AI reasoning                         │
│ 4. Interactive sidebar shows AI analysis                   │
└─────────────────────────────────────────────────────────────┘
  ↓
┌─────────────────────────────────────────────────────────────┐
│ LAWYER INTERACTION                                          │
│                                                             │
│ Lawyer clicks section P2:                                   │
│ → Sidebar shows all AI recommendations                      │
│ → Lawyer can:                                               │
│   [✓ Accept] → Apply redlining                             │
│   [✗ Reject] → Clear styling                               │
│   [✎ Modify] → Edit suggestion                             │
│   [+ Manual] → Add own redline                             │
│                                                             │
│ Final output:                                               │
│ {                                                           │
│   documentId: "doc_12345",                                 │
│   redlines: [                                               │
│     {                                                       │
│       sectionId: "p2",                                      │
│       action: "highlight",                                  │
│       status: "accepted",                                   │
│       aiRecommendation: true,                              │
│       lawyerNote: "Agreed - negotiate to $100k"            │
│     },                                                      │
│     {                                                       │
│       sectionId: "p7",                                      │
│       action: "strike-through",                            │
│       status: "manual",                                     │
│       aiRecommendation: false,                             │
│       lawyerNote: "Liability cap too low"                  │
│     }                                                       │
│   ],                                                        │
│   finalizedAt: "2025-11-13T19:00:00Z",                     │
│   finalizedBy: "lawyer@firm.com"                           │
│ }                                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Docling Integration Details

### How Docling Preserves PDF Integrity

**Key Concept:** Docling does NOT modify the PDF. It creates a separate JSON "map" of where everything is.

```
Original PDF (unchanged)          Docling JSON Map
┌─────────────────────┐          ┌──────────────────────────┐
│                     │          │ sections: [              │
│ 1. Payment Terms    │  ←───────│   {                      │
│                     │          │     id: "p1",            │
│ Client shall pay... │  ←───────│     bbox: {72,100,200,24}│
│                     │          │   },                     │
│ • Phase 1: ...      │  ←───────│   {                      │
│ • Phase 2: ...      │          │     id: "p2",            │
│ • Phase 3: ...      │          │     bbox: {72,140,468,60}│
│                     │          │   },                     │
│ ┌────┬────┬────┐    │  ←───────│   {                      │
│ │ A  │ B  │ C  │    │          │     id: "p3",            │
│ └────┴────┴────┘    │          │     bbox: {90,220,450,75}│
│                     │          │   }                      │
└─────────────────────┘          │ ]                        │
                                 └──────────────────────────┘
                                          ↓
                                 Feed to AI Agents
                                          ↓
                                 Overlay on PDF for visualization
```

### Docling Output Example (Real Structure)

```json
{
  "documentId": "doc_abc123",
  "filename": "service-agreement.pdf",
  "totalPages": 8,
  "sections": [
    {
      "id": "p1",
      "readingOrder": 1,
      "type": "section-header",
      "text": "1. PAYMENT TERMS",
      "bbox": {
        "x": 72,
        "y": 100,
        "width": 200,
        "height": 24,
        "page": 1
      },
      "confidence": 0.98,
      "fontSize": 16,
      "fontFamily": "Arial-Bold"
    },
    {
      "id": "p2",
      "readingOrder": 2,
      "type": "paragraph",
      "text": "Client shall pay Contractor the sum of One Hundred Twenty-Five Thousand Dollars ($125,000) for the services described in Exhibit A. Payment shall be made in three installments: (i) $40,000 upon execution of this Agreement, (ii) $40,000 upon completion of Phase 2, and (iii) $45,000 upon final delivery.",
      "bbox": {
        "x": 72,
        "y": 140,
        "width": 468,
        "height": 60,
        "page": 1
      },
      "confidence": 0.95,
      "wordCount": 47,
      "sentences": 2
    },
    {
      "id": "p3",
      "readingOrder": 3,
      "type": "bullet-list",
      "items": [
        {
          "text": "• Phase 1: Requirements gathering and project planning",
          "bbox": { "x": 90, "y": 220, "width": 450, "height": 18, "page": 1 }
        },
        {
          "text": "• Phase 2: Design, development, and initial testing",
          "bbox": { "x": 90, "y": 243, "width": 450, "height": 18, "page": 1 }
        },
        {
          "text": "• Phase 3: Final testing, deployment, and documentation",
          "bbox": { "x": 90, "y": 266, "width": 450, "height": 18, "page": 1 }
        }
      ],
      "text": "• Phase 1: Requirements gathering and project planning\n• Phase 2: Design, development, and initial testing\n• Phase 3: Final testing, deployment, and documentation",
      "bbox": {
        "x": 90,
        "y": 220,
        "width": 450,
        "height": 64,
        "page": 1
      },
      "confidence": 0.93,
      "itemCount": 3
    },
    {
      "id": "p4",
      "readingOrder": 4,
      "type": "table",
      "structure": {
        "rows": 4,
        "cols": 3,
        "hasHeader": true,
        "data": [
          ["Deliverable", "Timeline", "Cost"],
          ["Website Design & Development", "3 months", "$50,000"],
          ["Mobile Application", "4 months", "$75,000"],
          ["Total Project Cost", "-", "$125,000"]
        ]
      },
      "text": "Deliverable | Timeline | Cost\nWebsite Design & Development | 3 months | $50,000\nMobile Application | 4 months | $75,000\nTotal Project Cost | - | $125,000",
      "bbox": {
        "x": 72,
        "y": 320,
        "width": 468,
        "height": 120,
        "page": 1
      },
      "confidence": 0.89
    }
  ],
  "metadata": {
    "parsedAt": "2025-11-13T18:00:00Z",
    "doclingVersion": "1.0.0",
    "processingTime": 0.43,
    "averageConfidence": 0.94
  }
}
```

### Why This Structure is Perfect for AI Redlining

1. **Each section is discrete** → Can be analyzed independently by AI agents
2. **Bounding boxes are precise** → Can overlay exactly on PDF
3. **Type information** → AI agents know context (is this a table? bullet list?)
4. **Reading order** → AI agents understand document flow
5. **Original PDF unchanged** → Can always render pristine document

---

## AI Agent Integration

### Multi-Agent Analysis Architecture

```javascript
// backend/ai-agent-analyzer.js

class AIAgentAnalyzer {
    constructor() {
        this.agents = [
            new ComplianceAgent(),
            new RiskAssessmentAgent(),
            new FairnessAgent(),
            new StandardClauseAgent()
        ];
    }

    async analyzeDocument(doclingOutput) {
        const { sections } = doclingOutput;

        // Analyze all sections in parallel
        const analysisPromises = sections.map(section =>
            this.analyzeSection(section, sections)
        );

        const results = await Promise.all(analysisPromises);

        return this.aggregateResults(results);
    }

    async analyzeSection(section, allSections) {
        // Get context (previous/next sections for AI understanding)
        const context = this.buildContext(section, allSections);

        // Send to all AI agents in parallel
        const agentPromises = this.agents.map(agent =>
            agent.analyze(section, context)
        );

        const agentResults = await Promise.all(agentPromises);

        return {
            sectionId: section.id,
            recommendations: agentResults.filter(r => r.hasFindings)
        };
    }

    buildContext(section, allSections) {
        const index = allSections.findIndex(s => s.id === section.id);

        return {
            previousSection: index > 0 ? allSections[index - 1] : null,
            nextSection: index < allSections.length - 1 ? allSections[index + 1] : null,
            sectionIndex: index,
            totalSections: allSections.length,
            pageNumber: section.bbox.page
        };
    }

    aggregateResults(results) {
        // Group recommendations by section
        const redlineMap = {};

        results.forEach(result => {
            if (result.recommendations.length > 0) {
                redlineMap[result.sectionId] = result.recommendations;
            }
        });

        return redlineMap;
    }
}

// Example AI Agent (Risk Assessment)
class RiskAssessmentAgent {
    async analyze(section, context) {
        // Prepare prompt for AI (Claude, GPT, etc.)
        const prompt = `
You are a legal risk assessment AI agent reviewing a contract clause.

**Section Type:** ${section.type}
**Section Text:** ${section.text}

**Context:**
- Previous section: ${context.previousSection?.text || 'N/A'}
- Next section: ${context.nextSection?.text || 'N/A'}

**Task:** Analyze this clause for legal risks, unfavorable terms, missing protections.

**Output format (JSON):**
{
  "hasFindings": true/false,
  "findings": [
    {
      "severity": "high" | "medium" | "low",
      "action": "highlight" | "strike-through" | "comment" | "accept",
      "reason": "Brief explanation",
      "suggestion": "Specific recommendation",
      "confidence": 0.0 - 1.0
    }
  ]
}
        `.trim();

        // Call AI API (Claude, GPT-4, etc.)
        const response = await this.callAI(prompt);

        return {
            agent: 'risk-assessor',
            sectionId: section.id,
            ...response
        };
    }

    async callAI(prompt) {
        // Example: Call Claude API
        const response = await fetch('https://api.anthropic.com/v1/messages', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'x-api-key': process.env.ANTHROPIC_API_KEY,
                'anthropic-version': '2023-06-01'
            },
            body: JSON.stringify({
                model: 'claude-3-5-sonnet-20241022',
                max_tokens: 1024,
                messages: [{
                    role: 'user',
                    content: prompt
                }]
            })
        });

        const data = await response.json();

        // Parse AI response (assume it returns JSON)
        return JSON.parse(data.content[0].text);
    }
}
```

### AI Agent Output Example

```json
{
  "p2": [
    {
      "agent": "risk-assessor",
      "hasFindings": true,
      "findings": [
        {
          "severity": "high",
          "action": "highlight",
          "reason": "Payment amount ($125,000) is unusually high for the scope described. Industry standard for similar projects is $80,000-$100,000.",
          "suggestion": "Negotiate total cost down to $100,000, or ensure scope in Exhibit A justifies premium pricing.",
          "confidence": 0.87
        },
        {
          "severity": "medium",
          "action": "comment",
          "reason": "No late payment penalty or interest clause specified. Client could delay payment indefinitely without consequence.",
          "suggestion": "Add: 'Late payments shall accrue interest at 5% per annum from due date.'",
          "confidence": 0.91
        }
      ]
    },
    {
      "agent": "fairness-analyzer",
      "hasFindings": true,
      "findings": [
        {
          "severity": "medium",
          "action": "strike-through",
          "reason": "Payment schedule heavily favors client (32% upfront, 32% mid-project, 36% at end). Contractor bears significant cash flow risk.",
          "suggestion": "Rebalance to: 40% upfront, 40% at Phase 2 completion, 20% final delivery. This protects contractor cash flow.",
          "confidence": 0.82
        }
      ]
    },
    {
      "agent": "compliance-reviewer",
      "hasFindings": false,
      "findings": []
    }
  ],
  "p3": [
    {
      "agent": "risk-assessor",
      "hasFindings": true,
      "findings": [
        {
          "severity": "low",
          "action": "comment",
          "reason": "Phase timelines are vague. 'Phase 2 completion' is ambiguous - who determines completion?",
          "suggestion": "Add specific deliverables and acceptance criteria for each phase.",
          "confidence": 0.79
        }
      ]
    }
  ]
}
```

---

## Visual Redlining Rendering

### Frontend Architecture

```javascript
// frontend/redlining-renderer.js

class RedliningRenderer {
    constructor(pdfViewerCanvas, overlayCanvas) {
        this.pdfCanvas = pdfViewerCanvas;
        this.overlayCanvas = overlayCanvas;
        this.overlayCtx = overlayCanvas.getContext('2d');

        this.sections = [];
        this.redlineRecommendations = {};
        this.currentScale = 1.5; // PDF.js scale factor
    }

    async initialize(pdfUrl, doclingData, aiRecommendations) {
        // 1. Render PDF with PDF.js
        await this.renderPDF(pdfUrl);

        // 2. Store sections and recommendations
        this.sections = doclingData.sections;
        this.redlineRecommendations = aiRecommendations;

        // 3. Draw redline overlays
        this.drawRedlineOverlays();

        // 4. Setup interactive handlers
        this.setupInteractivity();
    }

    async renderPDF(pdfUrl) {
        const pdfjsLib = window['pdfjs-dist/build/pdf'];
        const loadingTask = pdfjsLib.getDocument(pdfUrl);
        const pdf = await loadingTask.promise;

        // For now, render page 1 (can extend to all pages)
        const page = await pdf.getPage(1);
        const viewport = page.getViewport({ scale: this.currentScale });

        this.pdfCanvas.width = viewport.width;
        this.pdfCanvas.height = viewport.height;
        this.overlayCanvas.width = viewport.width;
        this.overlayCanvas.height = viewport.height;

        const renderContext = {
            canvasContext: this.pdfCanvas.getContext('2d'),
            viewport: viewport
        };

        await page.render(renderContext).promise;
    }

    drawRedlineOverlays() {
        this.sections.forEach(section => {
            const recommendations = this.redlineRecommendations[section.id];

            if (recommendations && recommendations.length > 0) {
                this.drawSection(section, recommendations);
            } else {
                // No AI flags, draw light outline for reference
                this.drawSectionOutline(section, 'rgba(200, 200, 200, 0.3)');
            }
        });
    }

    drawSection(section, recommendations) {
        const bbox = this.transformBBox(section.bbox);

        // Determine color based on highest severity recommendation
        const highestSeverity = this.getHighestSeverity(recommendations);
        const style = this.getRedlineStyle(highestSeverity);

        // Draw background highlight
        this.overlayCtx.fillStyle = style.backgroundColor;
        this.overlayCtx.fillRect(bbox.x, bbox.y, bbox.width, bbox.height);

        // Draw border
        this.overlayCtx.strokeStyle = style.borderColor;
        this.overlayCtx.lineWidth = 2;
        this.overlayCtx.strokeRect(bbox.x, bbox.y, bbox.width, bbox.height);

        // Draw section label (P1, P2, P3...)
        this.overlayCtx.fillStyle = 'black';
        this.overlayCtx.font = 'bold 12px Arial';
        this.overlayCtx.fillText(
            section.id.toUpperCase(),
            bbox.x + 5,
            bbox.y + 15
        );

        // Draw severity indicator (icon)
        this.drawSeverityIcon(bbox, highestSeverity);

        // Store for click detection
        section._renderedBBox = bbox;
    }

    drawSectionOutline(section, color) {
        const bbox = this.transformBBox(section.bbox);

        this.overlayCtx.strokeStyle = color;
        this.overlayCtx.lineWidth = 1;
        this.overlayCtx.setLineDash([5, 5]);
        this.overlayCtx.strokeRect(bbox.x, bbox.y, bbox.width, bbox.height);
        this.overlayCtx.setLineDash([]);

        section._renderedBBox = bbox;
    }

    transformBBox(bbox) {
        // Docling bbox is in PDF coordinates
        // Need to transform to canvas coordinates (with scale)
        return {
            x: bbox.x * this.currentScale,
            y: bbox.y * this.currentScale,
            width: bbox.width * this.currentScale,
            height: bbox.height * this.currentScale
        };
    }

    getHighestSeverity(recommendations) {
        const severities = recommendations
            .flatMap(rec => rec.findings.map(f => f.severity));

        if (severities.includes('high')) return 'high';
        if (severities.includes('medium')) return 'medium';
        return 'low';
    }

    getRedlineStyle(severity) {
        const styles = {
            'high': {
                backgroundColor: 'rgba(255, 100, 100, 0.3)', // Red
                borderColor: 'rgba(255, 0, 0, 0.8)'
            },
            'medium': {
                backgroundColor: 'rgba(255, 200, 100, 0.3)', // Yellow
                borderColor: 'rgba(255, 165, 0, 0.8)'
            },
            'low': {
                backgroundColor: 'rgba(100, 200, 255, 0.2)', // Blue
                borderColor: 'rgba(0, 100, 255, 0.6)'
            }
        };

        return styles[severity] || styles['low'];
    }

    drawSeverityIcon(bbox, severity) {
        const icons = {
            'high': '⚠️',
            'medium': '⚡',
            'low': 'ℹ️'
        };

        this.overlayCtx.font = '16px Arial';
        this.overlayCtx.fillText(
            icons[severity],
            bbox.x + bbox.width - 25,
            bbox.y + 20
        );
    }

    setupInteractivity() {
        this.overlayCanvas.addEventListener('click', (e) => {
            const clickedSection = this.findSectionAtPoint(e.offsetX, e.offsetY);

            if (clickedSection) {
                this.showAnalysisSidebar(clickedSection);
            }
        });

        this.overlayCanvas.addEventListener('mousemove', (e) => {
            const hoveredSection = this.findSectionAtPoint(e.offsetX, e.offsetY);

            if (hoveredSection) {
                this.overlayCanvas.style.cursor = 'pointer';
                this.showTooltip(hoveredSection, e.offsetX, e.offsetY);
            } else {
                this.overlayCanvas.style.cursor = 'default';
                this.hideTooltip();
            }
        });
    }

    findSectionAtPoint(x, y) {
        return this.sections.find(section => {
            const bbox = section._renderedBBox;
            if (!bbox) return false;

            return x >= bbox.x && x <= bbox.x + bbox.width &&
                   y >= bbox.y && y <= bbox.y + bbox.height;
        });
    }

    showAnalysisSidebar(section) {
        const recommendations = this.redlineRecommendations[section.id];

        // Build sidebar HTML
        const sidebarHTML = `
            <div class="analysis-sidebar">
                <h3>Analysis for ${section.id.toUpperCase()}</h3>
                <p><strong>Type:</strong> ${section.type}</p>
                <p><strong>Text:</strong> ${section.text.substring(0, 100)}...</p>

                <div class="recommendations">
                    ${recommendations.map(rec => `
                        <div class="recommendation">
                            <h4>${rec.agent}</h4>
                            ${rec.findings.map(finding => `
                                <div class="finding ${finding.severity}">
                                    <span class="severity">${finding.severity.toUpperCase()}</span>
                                    <p><strong>Issue:</strong> ${finding.reason}</p>
                                    <p><strong>Suggestion:</strong> ${finding.suggestion}</p>
                                    <p><strong>Confidence:</strong> ${(finding.confidence * 100).toFixed(0)}%</p>

                                    <div class="actions">
                                        <button onclick="acceptRecommendation('${section.id}', '${finding.action}')">
                                            ✓ Accept
                                        </button>
                                        <button onclick="rejectRecommendation('${section.id}')">
                                            ✗ Reject
                                        </button>
                                        <button onclick="modifyRecommendation('${section.id}')">
                                            ✎ Modify
                                        </button>
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    `).join('')}
                </div>
            </div>
        `;

        document.getElementById('sidebar').innerHTML = sidebarHTML;
    }

    showTooltip(section, x, y) {
        const recommendations = this.redlineRecommendations[section.id];
        if (!recommendations) return;

        const tooltip = document.getElementById('tooltip');
        const summary = recommendations[0].findings[0]; // First finding

        tooltip.innerHTML = `
            <strong>${section.id.toUpperCase()}</strong><br>
            ${summary.severity.toUpperCase()}: ${summary.reason.substring(0, 80)}...
        `;

        tooltip.style.left = x + 15 + 'px';
        tooltip.style.top = y + 15 + 'px';
        tooltip.style.display = 'block';
    }

    hideTooltip() {
        document.getElementById('tooltip').style.display = 'none';
    }
}

// Initialize when page loads
document.addEventListener('DOMContentLoaded', async () => {
    const pdfCanvas = document.getElementById('pdf-canvas');
    const overlayCanvas = document.getElementById('overlay-canvas');

    const renderer = new RedliningRenderer(pdfCanvas, overlayCanvas);

    // Fetch data from backend
    const response = await fetch('/api/document/doc_12345/analysis');
    const data = await response.json();

    // Initialize renderer
    await renderer.initialize(
        data.pdfUrl,
        data.doclingOutput,
        data.aiRecommendations
    );
});
```

### HTML Structure

```html
<!DOCTYPE html>
<html>
<head>
    <title>AI Contract Redlining</title>
    <style>
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            display: flex;
        }

        #viewer-container {
            flex: 1;
            position: relative;
            overflow: auto;
        }

        #pdf-canvas, #overlay-canvas {
            position: absolute;
            top: 0;
            left: 0;
        }

        #sidebar {
            width: 400px;
            background: #f5f5f5;
            padding: 20px;
            overflow-y: auto;
            border-left: 1px solid #ccc;
        }

        .finding {
            margin: 10px 0;
            padding: 10px;
            border-radius: 5px;
            background: white;
            border-left: 4px solid;
        }

        .finding.high {
            border-left-color: #ff4444;
        }

        .finding.medium {
            border-left-color: #ffaa00;
        }

        .finding.low {
            border-left-color: #4444ff;
        }

        .actions button {
            margin: 5px;
            padding: 8px 12px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .actions button:hover {
            opacity: 0.8;
        }

        #tooltip {
            position: absolute;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 10px;
            border-radius: 5px;
            font-size: 12px;
            pointer-events: none;
            display: none;
            max-width: 300px;
            z-index: 1000;
        }
    </style>
</head>
<body>
    <div id="viewer-container">
        <canvas id="pdf-canvas"></canvas>
        <canvas id="overlay-canvas"></canvas>
        <div id="tooltip"></div>
    </div>

    <div id="sidebar">
        <h2>AI Analysis</h2>
        <p>Click on a highlighted section to see AI recommendations.</p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
    <script src="redlining-renderer.js"></script>
</body>
</html>
```

---

## Complete Code Implementation

### Backend API (Node.js + Python Docling Service)

**Python Service (Docling Parser):**
```python
# docling_service.py
from flask import Flask, request, jsonify
from docling.document_converter import DocumentConverter
import tempfile
import os

app = Flask(__name__)
converter = DocumentConverter()

@app.route('/parse', methods=['POST'])
def parse_document():
    """Parse PDF and return structured JSON with sections."""
    pdf_file = request.files.get('pdf')

    if not pdf_file:
        return jsonify({'error': 'No PDF file provided'}), 400

    # Save to temp file
    with tempfile.NamedTemporaryFile(delete=False, suffix='.pdf') as tmp:
        pdf_file.save(tmp.name)
        tmp_path = tmp.name

    try:
        # Parse with Docling
        result = converter.convert(tmp_path)
        doc_json = result.document.export_to_dict()

        # Transform to our format
        sections = transform_sections(doc_json)

        return jsonify({
            'success': True,
            'sections': sections,
            'metadata': {
                'totalPages': len(doc_json.get('pages', [])),
                'parsedAt': datetime.utcnow().isoformat(),
                'confidence': calculate_average_confidence(sections)
            }
        })
    except Exception as e:
        return jsonify({'error': str(e)}), 500
    finally:
        os.unlink(tmp_path)

def transform_sections(doc_json):
    """Transform Docling output to our section format."""
    sections = []
    reading_order = 1

    for page in doc_json.get('pages', []):
        page_num = page.get('page_number', 1)

        for elem in page.get('elements', []):
            sections.append({
                'id': f'p{reading_order}',
                'readingOrder': reading_order,
                'type': map_element_type(elem.get('type')),
                'text': elem.get('text', ''),
                'bbox': {
                    **elem.get('bbox', {}),
                    'page': page_num
                },
                'confidence': elem.get('confidence', 0.95),
                'metadata': {
                    'fontSize': elem.get('fontSize'),
                    'fontFamily': elem.get('fontFamily')
                }
            })
            reading_order += 1

    return sections

def map_element_type(docling_type):
    """Map Docling element types to our types."""
    type_map = {
        'heading': 'section-header',
        'paragraph': 'paragraph',
        'list': 'bullet-list',
        'table': 'table'
    }
    return type_map.get(docling_type, 'paragraph')

def calculate_average_confidence(sections):
    """Calculate average confidence across all sections."""
    if not sections:
        return 0.0

    total = sum(s.get('confidence', 0) for s in sections)
    return total / len(sections)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Node.js API (Express):**
```javascript
// server.js
const express = require('express');
const multer = require('multer');
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');
const path = require('path');

const app = express();
const upload = multer({ dest: 'uploads/' });

// Import AI agent analyzer
const AIAgentAnalyzer = require('./ai-agent-analyzer');
const analyzer = new AIAgentAnalyzer();

// Endpoint: Upload and analyze document
app.post('/api/document/upload', upload.single('pdf'), async (req, res) => {
    try {
        const pdfPath = req.file.path;

        // 1. Send to Docling for parsing
        console.log('Parsing document with Docling...');
        const doclingOutput = await parseWithDocling(pdfPath);

        // 2. Send to AI agents for analysis
        console.log('Analyzing with AI agents...');
        const aiRecommendations = await analyzer.analyzeDocument(doclingOutput);

        // 3. Save document and analysis
        const documentId = generateDocumentId();
        const pdfUrl = await savePDF(pdfPath, documentId);

        await saveAnalysis(documentId, {
            doclingOutput,
            aiRecommendations,
            uploadedAt: new Date().toISOString(),
            uploadedBy: req.body.userId || 'anonymous'
        });

        // 4. Return results
        res.json({
            success: true,
            documentId,
            pdfUrl,
            sections: doclingOutput.sections,
            aiRecommendations,
            metadata: doclingOutput.metadata
        });

        // Cleanup
        fs.unlinkSync(pdfPath);
    } catch (error) {
        console.error('Document processing error:', error);
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
});

// Endpoint: Get document analysis
app.get('/api/document/:documentId/analysis', async (req, res) => {
    try {
        const { documentId } = req.params;
        const analysis = await loadAnalysis(documentId);

        if (!analysis) {
            return res.status(404).json({ error: 'Document not found' });
        }

        res.json({
            success: true,
            documentId,
            pdfUrl: `/pdfs/${documentId}.pdf`,
            ...analysis
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Endpoint: Accept/reject AI recommendation
app.post('/api/document/:documentId/redline/:sectionId', async (req, res) => {
    try {
        const { documentId, sectionId } = req.params;
        const { action, status, note } = req.body;

        await saveRedlineDecision(documentId, sectionId, {
            action,
            status, // 'accepted', 'rejected', 'modified'
            note,
            decidedAt: new Date().toISOString(),
            decidedBy: req.body.userId || 'anonymous'
        });

        res.json({ success: true });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Helper: Call Docling service
async function parseWithDocling(pdfPath) {
    const formData = new FormData();
    formData.append('pdf', fs.createReadStream(pdfPath));

    const response = await axios.post('http://localhost:5000/parse', formData, {
        headers: formData.getHeaders(),
        timeout: 30000 // 30 second timeout
    });

    return response.data;
}

// Helper: Generate unique document ID
function generateDocumentId() {
    return 'doc_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
}

// Helper: Save PDF to storage
async function savePDF(tempPath, documentId) {
    const pdfDir = path.join(__dirname, 'public', 'pdfs');
    if (!fs.existsSync(pdfDir)) {
        fs.mkdirSync(pdfDir, { recursive: true });
    }

    const finalPath = path.join(pdfDir, `${documentId}.pdf`);
    fs.copyFileSync(tempPath, finalPath);

    return `/pdfs/${documentId}.pdf`;
}

// Helper: Save analysis to database (simplified - use real DB in production)
async function saveAnalysis(documentId, analysis) {
    const analysisDir = path.join(__dirname, 'data', 'analyses');
    if (!fs.existsSync(analysisDir)) {
        fs.mkdirSync(analysisDir, { recursive: true });
    }

    const filePath = path.join(analysisDir, `${documentId}.json`);
    fs.writeFileSync(filePath, JSON.stringify(analysis, null, 2));
}

async function loadAnalysis(documentId) {
    const filePath = path.join(__dirname, 'data', 'analyses', `${documentId}.json`);

    if (!fs.existsSync(filePath)) {
        return null;
    }

    return JSON.parse(fs.readFileSync(filePath, 'utf-8'));
}

async function saveRedlineDecision(documentId, sectionId, decision) {
    const decisionsDir = path.join(__dirname, 'data', 'redlines');
    if (!fs.existsSync(decisionsDir)) {
        fs.mkdirSync(decisionsDir, { recursive: true });
    }

    const filePath = path.join(decisionsDir, `${documentId}.json`);

    let decisions = {};
    if (fs.existsSync(filePath)) {
        decisions = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    }

    decisions[sectionId] = decision;
    fs.writeFileSync(filePath, JSON.stringify(decisions, null, 2));
}

// Serve static files
app.use(express.static('public'));

app.listen(3000, () => {
    console.log('Server running on http://localhost:3000');
    console.log('Ensure Docling service is running on http://localhost:5000');
});
```

---

## Universal Document Support

### Document Conversion Layer

```javascript
// document-converter.js
const libre = require('libreoffice-convert');
const fs = require('fs');
const path = require('path');

class UniversalDocumentConverter {
    async convertToPDF(inputPath) {
        const ext = path.extname(inputPath).toLowerCase();

        // If already PDF, return as-is
        if (ext === '.pdf') {
            return inputPath;
        }

        // Convert to PDF based on format
        switch (ext) {
            case '.docx':
            case '.doc':
            case '.rtf':
            case '.odt':
                return await this.convertOfficeToPDF(inputPath);

            case '.pptx':
            case '.ppt':
            case '.odp':
                return await this.convertOfficeToPDF(inputPath);

            case '.html':
            case '.htm':
                return await this.convertHTMLToPDF(inputPath);

            case '.md':
            case '.markdown':
                return await this.convertMarkdownToPDF(inputPath);

            default:
                throw new Error(`Unsupported format: ${ext}`);
        }
    }

    async convertOfficeToPDF(inputPath) {
        return new Promise((resolve, reject) => {
            const inputBuf = fs.readFileSync(inputPath);

            libre.convert(inputBuf, '.pdf', undefined, (err, pdfBuf) => {
                if (err) return reject(err);

                const outputPath = inputPath.replace(path.extname(inputPath), '.pdf');
                fs.writeFileSync(outputPath, pdfBuf);

                resolve(outputPath);
            });
        });
    }

    async convertHTMLToPDF(inputPath) {
        // Use Puppeteer for HTML → PDF
        const puppeteer = require('puppeteer');

        const browser = await puppeteer.launch({ headless: true });
        const page = await browser.newPage();

        const htmlContent = fs.readFileSync(inputPath, 'utf-8');
        await page.setContent(htmlContent);

        const outputPath = inputPath.replace(path.extname(inputPath), '.pdf');
        await page.pdf({ path: outputPath, format: 'Letter' });

        await browser.close();

        return outputPath;
    }

    async convertMarkdownToPDF(inputPath) {
        // Use pandoc for Markdown → PDF
        const { exec } = require('child_process');
        const outputPath = inputPath.replace(path.extname(inputPath), '.pdf');

        return new Promise((resolve, reject) => {
            exec(`pandoc "${inputPath}" -o "${outputPath}"`, (error) => {
                if (error) return reject(error);
                resolve(outputPath);
            });
        });
    }
}

module.exports = UniversalDocumentConverter;
```

**Updated Upload Endpoint:**
```javascript
app.post('/api/document/upload', upload.single('file'), async (req, res) => {
    try {
        const originalPath = req.file.path;
        const converter = new UniversalDocumentConverter();

        // 1. Convert to PDF if needed
        console.log('Converting document to PDF...');
        const pdfPath = await converter.convertToPDF(originalPath);

        // 2. Parse with Docling
        const doclingOutput = await parseWithDocling(pdfPath);

        // 3. AI analysis
        const aiRecommendations = await analyzer.analyzeDocument(doclingOutput);

        // ... rest of the flow ...
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

---

## Deployment Architecture

### Docker Compose Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Python Docling Service
  docling-service:
    build:
      context: ./docling-service
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=production
    volumes:
      - ./temp:/app/temp
    restart: unless-stopped

  # Node.js API + Frontend
  web-app:
    build:
      context: ./web-app
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DOCLING_SERVICE_URL=http://docling-service:5000
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    volumes:
      - ./uploads:/app/uploads
      - ./data:/app/data
      - ./public:/app/public
    depends_on:
      - docling-service
    restart: unless-stopped

  # Optional: Database (PostgreSQL for production)
  database:
    image: postgres:15
    environment:
      - POSTGRES_DB=redlining
      - POSTGRES_USER=redline_user
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres-data:
```

**Docling Service Dockerfile:**
```dockerfile
# docling-service/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "--timeout", "120", "docling_service:app"]
```

**Web App Dockerfile:**
```dockerfile
# web-app/Dockerfile
FROM node:18

WORKDIR /app

# Install system dependencies for LibreOffice conversion
RUN apt-get update && apt-get install -y \
    libreoffice \
    libreoffice-writer \
    && rm -rf /var/lib/apt/lists/*

# Install Node dependencies
COPY package*.json ./
RUN npm install --production

# Copy application
COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

---

## Complete Example Walkthrough

### Step-by-Step: From Upload to Redlined PDF

**1. User Uploads Contract (DOCX)**
```
User action: Upload "service-agreement.docx"
Frontend: POST /api/document/upload
```

**2. Backend Converts DOCX → PDF**
```javascript
// server.js
const converter = new UniversalDocumentConverter();
const pdfPath = await converter.convertToPDF('service-agreement.docx');
// Output: service-agreement.pdf
```

**3. Docling Parses PDF Structure**
```python
# docling_service.py
result = converter.convert('service-agreement.pdf')
sections = transform_sections(result.document.export_to_dict())

# Output:
{
  "sections": [
    { "id": "p1", "type": "section-header", "text": "1. PAYMENT TERMS", ... },
    { "id": "p2", "type": "paragraph", "text": "Client shall pay...", ... },
    { "id": "p3", "type": "bullet-list", "items": [...], ... },
    { "id": "p4", "type": "table", "structure": {...}, ... }
  ]
}
```

**4. AI Agents Analyze Each Section**
```javascript
// ai-agent-analyzer.js
const aiRecommendations = await analyzer.analyzeDocument(doclingOutput);

// For section P2:
// - Compliance Agent → No issues
// - Risk Agent → High severity: Payment amount too high
// - Fairness Agent → Medium severity: Payment schedule favors client

// Output:
{
  "p2": [
    {
      "agent": "risk-assessor",
      "findings": [
        {
          "severity": "high",
          "action": "highlight",
          "reason": "Payment amount ($125k) unusually high...",
          "suggestion": "Negotiate to $100k"
        }
      ]
    }
  ]
}
```

**5. Frontend Renders PDF with AI Overlays**
```javascript
// redlining-renderer.js
await renderer.initialize(
    '/pdfs/doc_12345.pdf',     // Original PDF
    doclingOutput,              // Sections with coordinates
    aiRecommendations           // AI analysis
);

// Visual result:
// - PDF rendered pixel-perfect
// - Section P2 highlighted in red (high severity)
// - Tooltip shows AI reasoning
// - Sidebar shows detailed recommendations
```

**6. Lawyer Interacts**
```
User action: Click on section P2
Sidebar displays:
  - Risk Agent: Payment too high → Negotiate to $100k
  - Fairness Agent: Payment schedule unfavorable

Lawyer clicks: [Accept]
  → Frontend sends: POST /api/document/doc_12345/redline/p2
  → Backend saves decision: { action: "highlight", status: "accepted" }
  → PDF overlay updates: P2 now marked as "accepted redline"
```

**7. Final Export**
```javascript
// Export redlined contract
const redlines = await loadRedlineDecisions('doc_12345');

// Generate final PDF with annotations
// OR export as Word document with track changes
// OR export as JSON for contract management system
```

---

## Summary: How Docling Enables End-to-End AI Redlining

### The Complete Flow

1. **Universal Document Input** → Any format (PDF, DOCX, PPTX, etc.)
2. **Docling Parsing** → Structure + coordinates extracted
3. **AI Agent Analysis** → Each section analyzed by multiple agents
4. **Visual Rendering** → Original PDF with AI recommendation overlays
5. **Lawyer Review** → Accept/reject/modify AI suggestions
6. **Final Output** → Redlined contract with tracked changes

### Why This Works

| Component | Role | Why Docling is Perfect |
|-----------|------|----------------------|
| **Document Parsing** | Extract structure | Docling gives paragraphs, tables, bullets with types |
| **Coordinate Mapping** | Position on PDF | Docling provides native bounding boxes |
| **AI Input** | Feed to agents | Each Docling section is discrete, analyzable chunk |
| **Visualization** | Overlay on PDF | Docling bbox maps exactly to PDF canvas coordinates |
| **Interactivity** | Click sections | Docling IDs (P1, P2, P3) are stable references |

### The Key Insight

**Docling creates a "structural map" of the document. This map enables:**
- ✅ **AI agents** to analyze each structural element independently
- ✅ **Visual rendering** to overlay precisely on the original PDF
- ✅ **Human interaction** to reference sections unambiguously

**The PDF never changes. Docling just tells you WHERE everything is and WHAT it is.**

This is exactly what you need for professional contract redlining with AI assistance.

---

## Next Steps

**Immediate Action:**
1. Run Docling POC tomorrow (test on your PDFs)
2. Validate structure extraction quality
3. If successful → Implement full pipeline

**Implementation Timeline:**
- Week 1: Docling POC + validation
- Week 2: AI agent integration
- Week 3: Frontend rendering
- Week 4: Production deployment

**Total: 4 weeks to full AI-driven redlining system with universal document support.**

---

**This is the complete architecture. Docling is the missing piece that makes AI-powered contract redlining practical and production-ready.**
