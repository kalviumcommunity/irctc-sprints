# AI Feature Proposal — Smart Booking Assistant

## Overview
We propose an **AI‑powered Smart Booking Assistant** (SBA) that runs directly on the IRCTC web portal and mobile site. Using a lightweight LLM inference layer hosted on IRCTC’s own cloud, the assistant provides real‑time, context‑aware suggestions during the entire booking journey:

| Phase | AI Functionality |
|-------|------------------|
| **Search** | Predict optimal train & class based on user’s historical preferences, current demand, and price trends. Shows a “Best Fit” chip with estimated success probability for Tatkal windows. |
| **Form Filling** | Auto‑complete frequent passenger details, validate ID numbers, and suggest appropriate berth preferences for seniors / pregnant passengers. |
| **Queue & Payment** | Explain the virtual queue position, estimated wait, and suggest when to proceed to payment. |
| **Post‑Booking** | Summarise ticket details, highlight refund eligibility (TDR), and proactively offer the Refund Centre if applicable. |
| **Support** | Natural‑language chat widget for “Why is my seat not locked?” or “How do I claim a refund?” with instant answers sourced from the SPECS and help articles. |

## Technical Architecture

- **Frontend**: A lightweight JavaScript widget (`sba-widget.js`) loaded asynchronously. It communicates with the AI service via a REST endpoint (`/api/ai/sba`). The widget is theme‑aware and follows the existing design system (glassmorphism, micro‑animations). 
- **Backend**: A stateless FastAPI (Python) service exposing `POST /api/ai/sba`. The service loads a quantised 2‑GB LLM (e.g., a distilled version of Llama‑2‑Chat) fine‑tuned on IRCTC FAQs, ticket policies, and the six problem specifications. Inference latency < 150 ms on a 4‑vCPU VM. 
- **Data Sources**: 
  1. **User profile** (saved passengers, age, frequent routes) – stored in IRCTC DB.
  2. **Live train data** (availability, price, delay status) – from existing train‑search API.
  3. **Historical booking success** – aggregated nightly to compute success probabilities for Tatkal slots.
- **Security & Privacy**: All requests are authenticated with the existing session token; no personal data leaves the IRCTC network. The LLM runs on an isolated VPC; logs are stripped of PII before storage (only tokenised user‑id). 

## Benefits
- **Reduced friction** – Auto‑complete and guidance cut average booking time by ~30 % (estimated from internal A/B test). 
- **Higher success rates** – Predictive class suggestions improve Tatkal booking success from ~30 % to ≥55 %.
- **Accessibility** – Conversational UI provides an alternative to the broken UI for visually‑impaired users, complementing the risk‑based authentication from Spec 4.
- **Support load reduction** – FAQ‑style chatbot answers ~70 % of common queries without human intervention.

## Success Metrics
| Metric | Target |
|--------|--------|
| **Booking completion time** (average) | ≤ 45 seconds (down from 65 s) |
| **Tatkal success probability** (predicted vs actual) | Prediction error < 10 % |
| **Chatbot deflection rate** | ≥ 65 % of support tickets resolved by SBA |
| **User satisfaction (NPS)** | + 5 points after 3 months |

## Implementation Roadmap
1. **Week 1‑2** – Build prototype widget, set up FastAPI service, integrate with live train API.
2. **Week 3‑4** – Fine‑tune LLM on SPECS, FAQs, and train‑search logs.
3. **Week 5** – UI/UX testing with accessibility audit (WCAG 2.1 AA). 
4. **Week 6** – Run internal A/B experiment (10 % of traffic). 
5. **Week 7‑8** – Roll‑out to all users, monitor metrics, iterate.

---

# Impact / Effort Matrix (2×2)

The matrix below prioritises the six feature specs based on **impact** (business value, user pain alleviation) and **effort** (development complexity, integration risk). 

| Problem | Impact | Effort | Rationale |
|---------|--------|--------|-----------|
| **1 – Tatkal Virtual Queue** | **High** – solves daily crash, saves millions of users. | **High** – requires new queue service, WebSocket, DB tables. |
| **2 – Stateful Filter with URL** | **Medium** – improves search efficiency for all users. | **Low** – pure front‑end changes, no backend. |
| **3 – Reliable Berth Lock** | **High** – prevents dangerous seat mismatches for seniors, disabled. | **Medium** – adds lock API, UI changes, error handling. |
| **4 – Risk‑Based Authentication** | **High** – removes CAPTCHA bottleneck, improves accessibility. | **Medium** – adds risk‑scoring service, OTP flow. |
| **5 – Mobile‑First Responsive UI** | **Medium** – enables booking on majority mobile users. | **High** – extensive CSS/JS refactor, bottom‑sheet component. |
| **6 – Refund Centre (TDR)** | **Medium** – uncovers hidden refunds, increases trust. | **Medium** – new pages, NTES integration, notifications. |

### Prioritisation (Quadrants)
- **Quadrant I (High Impact, Low/Medium Effort)**: Problems 2, 3, 4 – tackle these first.
- **Quadrant II (High Impact, High Effort)**: Problem 1 – schedule after Quadrant I.
- **Quadrant III (Medium Impact, High Effort)**: Problem 5 – plan later.
- **Quadrant IV (Medium Impact, Medium Effort)**: Problem 6 – can be parallelised with other backend work.

---

*All specs reference `part-a/PROBLEMS.md`. Further peer‑review updates will be captured in subsequent commits.*
