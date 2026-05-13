# AI-Powered Invoice Reminder Workflow with n8n + Gemini

This project automates overdue invoice reminders using n8n, Google Sheets, Gmail, and Gemini AI. The workflow generates personalized reminder emails, routes them through a human approval step, and sends them sequentially to customers.

## Features
- Reads overdue invoices from Google Sheets
- Uses Gemini AI to generate personalized email reminders
- Structured JSON output parsing
- Human approval workflow using Gmail Send & Wait
- Sequential email processing with Loop Over Items
- Dynamic email generation per customer

## Tech Stack
- n8n
- Google Gemini API
- Gmail API
- Google Sheets API
- Structured Output Parser

## Workflow Architecture
1. Pull overdue invoices from Google Sheets
2. Aggregate invoice records
3. Generate AI email drafts
4. Parse structured JSON output
5. Split customer records into individual items
6. Human approval step
7. Send approved emails

# Main Workflow Image 
![Workflow Screenshot](screenshots/MainWorkflow.png)
# Sub Workflow Image
![Subworkflow Screenshot](screenshots/SubWorkflow.png)
# Approval Email
![Approval Email](screenshots/ApprovalEmail.png)
# Output 
![Output](screenshots/Output.png)

# Main Technical Issues Encountered

## 1. Structured Output Parser Schema Mismatches

The Gemini model output did not consistently match the JSON schema required by the n8n Structured Output Parser.

### Causes

* Nested arrays/objects were defined incorrectly
* Required fields did not align with generated output
* Schema structure changed during iteration (`email`, `invoiceID`, etc.)
* AI output sometimes wrapped objects differently than expected
* Multiple competing schema definitions existed between:

  * AI prompt
  * parser node
  * downstream workflow assumptions

### Symptoms

* `Model output doesn't fit required format`
* `missing field`
* empty parser responses
* malformed JSON
* duplicated/nested `output` objects

### Resolution

* Simplified the schema structure
* Removed redundant schema instructions from the AI prompt
* Centralized validation in the Structured Output Parser only
* Reduced nesting complexity
* Incrementally tested schema changes with small datasets

---

# 2. Array vs Single Item Data Handling in n8n

The workflow initially processed arrays incorrectly, causing repeated emails, duplicate items, and undefined fields.

### Causes

* AI returned:

```json id="jlwm99"
{
  "customers": [...]
}
```

instead of individual items

* Gmail and Send & Wait nodes expect one item per execution
* Arrays were passed directly into nodes requiring flattened records

### Symptoms

* only first email sent
* duplicated processing
* 25 generated items instead of 5
* `[object Object]`
* undefined expressions

### Resolution

* Used `Split Out` node to flatten customer arrays into individual items
* Introduced `Loop Over Items` with batch size = 1
* Ensured sequential item processing

---

# 3. Loop Over Items Configuration Problems

The loop initially restarted instead of advancing.

### Causes

* Workflow connected back into the main loop input rather than the continuation input
* Additional bypass connections skipped loop control logic

### Symptoms

* only first item processed repeatedly
* workflow stopped after first approval
* infinite/repeated execution behavior

### Resolution

* Rewired workflow into the loop continuation connector
* Removed direct bypass connections
* Used sequential iteration with batch size = 1

---

# 4. Send & Wait Payload Replacement

`Send and Wait` replaced the original workflow payload with approval metadata.

### Causes

After approval, the node output became:

```json id="jlwm9a"
{
  "data": {
    "approved": true
  }
}
```

instead of preserving customer/email data.

### Symptoms

* Gmail subject became `(no subject)`
* fields evaluated as null/undefined
* `Cannot read properties of undefined/null (reading 'trim')`

### Resolution

* Added a `Set` node before `Send & Wait`
* Persisted required fields manually
* Referenced preserved node data explicitly in downstream nodes

---

# 5. Subworkflow Data Passing Failures

The child workflow initially received no input data.

### Causes

The `Execute Workflow` node was not configured to pass input items into the subworkflow.

### Symptoms

* all values became null
* subworkflow executed with empty payloads
* “No fields - node executed, but no items were sent on this branch”

### Resolution

* Enabled input passthrough in `Execute Workflow`
* Verified child workflow item structure before processing

---

# 6. Gemini API Operational Issues

The Gemini API produced intermittent failures during development.

### Causes

* free-tier quota exhaustion
* invalid/unsupported model names
* transient API/network failures
* overly large prompts/schema payloads

### Symptoms

* HTTP 429 quota errors
* `fetch failed`
* empty responses
* inconsistent execution behavior

### Resolution

* Reduced prompt complexity
* Limited test rows during development
* Switched to supported Gemini models
* Simplified structured outputs
* Retried executions after quota reset windows





