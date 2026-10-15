# AI-Powered Lead Qualification System

An AI-powered lead management system built with **n8n, Tally, Airtable, and Google Sheets** to automate customer inquiry capture, knowledge retrieval, and lead management for an electronics business.

## Overview

The Lead Qualification System streamlines how an electronics business receives, records, and responds to customer inquiries. It integrates a Tally form for customer submissions, Airtable for centralized lead storage, Google Sheets as the business knowledge base, and n8n to orchestrate the workflow.

The AI agent retrieves approved information about products, prices, stock availability, warranties, and company policies to provide consistent, knowledge-based responses.

Every inquiry submitted through Tally is intended to be recorded in Airtable, regardless of its subject or whether the AI agent can answer it.

## Objectives

- Automate customer inquiry capture and lead logging.
- Reduce manual data entry and repetitive customer support tasks.
- Retrieve accurate business information from a centralized knowledge base.
- Improve response consistency and efficiency.
- Maintain structured customer inquiry records for follow-up.
- Apply guardrails to prevent unsupported answers and unrelated recommendations.

## Technology Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and AI agent orchestration |
| Tally | Customer inquiry form and data collection |
| Airtable | Lead database and inquiry management |
| Google Sheets | Business knowledge base |
| AI Agent | Inquiry interpretation and knowledge-based responses |

## System Architecture

```text
Customer
   |
   v
Tally Form
   |
   v
n8n Workflow
   |
   +--------------------------+
   |                          |
   v                          v
Airtable                  AI Agent
Lead Logging                  |
                              v
                       Google Sheets
                       Knowledge Base
                              |
                              v
                       Customer Response
```

The workflow can be configured to log every Tally submission directly to Airtable before passing the inquiry to the AI agent. This reduces dependence on the AI model remembering to invoke the logging tool.

## Workflow Components

### 1. Tally Form — Customer Data Collection

The Tally form collects the following information:

- **Name:** Customer's name.
- **Email:** Customer's email address.
- **Phone Number:** Customer's contact number.
- **Inquiries:** The customer's question, request, or feedback.

The submitted information is passed to the n8n workflow for processing.

### 2. Airtable — Lead Management

Airtable serves as the centralized database for customer inquiries.

**Base:** `Leads`

**Table:** `Customer Data`

| Airtable Field | Source |
|---|---|
| Name | Tally Name |
| Email | Tally email |
| Phone Number | Tally phone number |
| Inquiries | Tally inquiries |

Each distinct Tally submission should create a corresponding Airtable record.

The workflow should verify successful record creation and handle failures without falsely reporting success. Duplicate submissions should be managed using a stable submission identifier where available.

### 3. Google Sheets — Business Knowledge Base

Google Sheets stores the approved information used by the AI agent to answer customer questions.

The sample knowledge base can contain the following worksheets:

| Worksheet | Purpose |
|---|---|
| Products | Product descriptions, brands, stock quantities, and warranties |
| Price_List | Current selling prices and approved discounts |
| Company_Policy | Returns, payments, deliveries, and other company policies |
| FAQs | Common questions and approved answers |
| Sales | Sample transaction and order information |
| Purchases | Supplier purchase records |
| Suppliers | Supplier reference information |

The AI agent should retrieve the relevant records before providing company-specific answers.

### 4. AI Agent — Inquiry Handling

The AI agent interprets customer inquiries and uses the Google Sheets tool to retrieve relevant business information.

Its responsibilities include:

- Answering product and pricing questions.
- Checking recorded stock availability.
- Explaining documented warranties and company policies.
- Handling inquiries using verified knowledge-base information.
- Explaining when information is unavailable.
- Keeping responses within the defined electronics business scope.

The agent must not invent prices, stock quantities, product specifications, or policies.

## System Guardrails

The system is designed to follow these rules:

1. Every Tally submission must be processed for Airtable logging.
2. Customer details must be mapped accurately to the corresponding Airtable fields.
3. Google Sheets is the source of truth for company-specific information.
4. Only active price records should be used when quoting current prices.
5. Missing or conflicting information must not be fabricated.
6. The agent must not recommend unrelated services, providers, or products outside its defined scope.
7. Customer information must remain confidential.
8. Existing lead records and knowledge-base data must not be modified without authorization.
9. The system must not claim that an operation succeeded unless success is confirmed.
10. Failed logging operations must be surfaced for recovery.

## Setup Instructions

### Prerequisites

Before configuring the workflow, ensure you have:

- An n8n instance.
- A Tally account and published form.
- An Airtable account with a `Leads` base and `Customer Data` table.
- A Google Sheets knowledge base populated with approved business information.
- An AI model supported by your n8n AI Agent node.
- The required credentials and permissions for each integration.

### Step 1: Configure Tally

Create a form containing:

- Name
- Email
- Phone Number
- Inquiries

Connect the form to an n8n Tally Trigger or the appropriate Tally submission integration.

### Step 2: Configure Airtable

Create the `Leads` base and `Customer Data` table.

Ensure that the table contains the four fields specified above. Configure the Airtable integration in n8n with permission to create records.

### Step 3: Configure Google Sheets

Upload or create the business knowledge base in Google Sheets.

Keep each information category in its own worksheet and ensure that column headers are clear and consistent.

Configure the Google Sheets tool to retrieve the relevant rows from the appropriate worksheet.

### Step 4: Configure the AI Agent

Add an AI Agent node in n8n and configure the supported language model.

Provide the system prompt defining the agent's role, available tools, mandatory lead-logging rules, knowledge retrieval procedures, and business guardrails.

Connect the Google Sheets knowledge retrieval tool to the agent.

If Airtable logging is performed through an AI Agent tool, configure the workflow carefully to ensure that the tool is invoked for every Tally submission. For stronger reliability, use a dedicated Airtable Create Record node before the AI Agent.

### Step 5: Connect the Workflow

A recommended workflow is:

```text
Tally Trigger
     |
     v
Validate and Map Form Fields
     |
     v
Airtable: Create Lead Record
     |
     v
Check Logging Result
     |
     v
AI Agent
     |
     v
Google Sheets: Retrieve Knowledge
     |
     v
Generate Response
```

Configure error handling so that failed Airtable operations are reported and the original submission remains available for recovery.

### Step 6: Test the Workflow

Submit test inquiries through Tally and confirm that the workflow creates the correct Airtable records.

Test product questions against the Google Sheets knowledge base and verify that responses match the stored information.

## Testing Scenarios

| Test Case | Expected Result |
|---|---|
| Customer asks for a product price | Retrieve the active price from Google Sheets |
| Customer asks about stock | Retrieve the recorded stock quantity and status |
| Customer asks about warranty | Return the documented warranty period |
| Customer asks about returns | Retrieve the applicable company policy |
| Customer asks an unrelated question | Log the inquiry and politely explain the agent's scope |
| Customer submits incomplete information | Preserve supplied values and follow validation rules |
| Airtable returns an error | Report the failure and preserve the submission for recovery |
| Same submission is processed again | Prevent duplicate creation where idempotency is configured |
| Product information is missing | Explain that the information requires confirmation |

## Security and Privacy

- Store integration credentials in n8n's credential manager.
- Apply least-privilege access to Airtable and Google Sheets.
- Avoid exposing customer information in logs or AI prompts unnecessarily.
- Do not collect passwords, PINs, or one-time passcodes.
- Restrict access to customer records to authorized personnel.
- Use appropriate error handling and data-retention practices.

## Current Scope and Future Enhancements

The current design focuses on automated inquiry capture, knowledge retrieval, and structured lead management.

Potential future enhancements include:

- Lead scoring based on purchase intent and urgency.
- Lead categorization by product interest.
- Automated assignment of leads to sales representatives.
- Email notifications for new inquiries.
- Follow-up reminders for unanswered leads.
- Sales pipeline dashboards and conversion reports.
- Duplicate detection using Tally submission IDs.
- Integration with CRM platforms.

These enhancements should be treated as future work unless implemented in the workflow.

## Expected Benefits

- Less manual handling of customer inquiries.
- Consistent access to approved business information.
- Centralized and structured lead records.
- Faster retrieval of product and pricing information.
- Improved traceability of customer inquiries.
- A foundation for future lead qualification and sales automation.

## Disclaimer

This project is a demonstration of AI-assisted business process automation. Product records, prices, supplier information, and company policies used in the sample knowledge base are fictional and should be replaced with verified business data before production use.

---

**Project:** AI-Powered Lead Qualification System  
**Workflow platform:** n8n  
**Data collection:** Tally  
**Lead database:** Airtable  
**Knowledge base:** Google Sheets
