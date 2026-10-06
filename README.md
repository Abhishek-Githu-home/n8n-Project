# Bulk Job Application Sender

![n8n Workflow Architecture](https://github.com/Abhishek-Githu-home/n8n-Project/raw/main/workflow-diagram.png)

A powerful n8n workflow that automates the job-application process by:

- accepting a JD PDF and a candidate resume PDF through a form,
- extracting text from both files,
- matching the candidate profile against job requirements,
- filtering out unsuitable roles using hard/soft blocker logic,
- and sending recruiter emails automatically through Gmail.

This project is designed for QA and automation-focused job seekers who want to apply to multiple roles faster while keeping the process controlled and customizable.

---

## Overview

This workflow is built in n8n and acts as a smart bulk outreach system for recruitment.

The idea is simple:

1. The user submits a PDF with multiple job descriptions.
2. The user uploads their resume PDF.
3. The workflow extracts text from both documents.
4. Matching logic compares the resume and JD keywords.
5. The candidate's experience, skills, and role fit are checked.
6. Suitable jobs are shortlisted.
7. A recruiter email is generated and sent using Gmail.

The workflow supports both:

- Preview mode: sends all emails to the candidate's inbox for testing
- Live mode: sends emails to recruiters

---

## Workflow Flow

```
┌─────────────────────┐
│ Application Form    │ (User uploads JD PDF + Resume PDF)
└──────────┬──────────┘
           │
           ├────────────────────────────┐
           │                            │
           ▼                            ▼
    ┌─────────────┐            ┌──────────────────┐
    │Read Resume  │            │ Read Job Desc.   │
    │Extract Text │            │ Extract Text     │
    └──────┬──────┘            └────────┬─────────┘
           │                            │
           └─────────┬──────────────────┘
                     │
                     ▼
         ┌──────────────────────────────┐
         │ Match Jobs & Prepare Emails  │
         │ (Smart Filtering Logic)      │
         └──────────────┬───────────────┘
                        │
                        ▼
         ┌──────────────────────────────┐
         │ Send Application Email       │
         │ (Via Gmail)                  │
         └──────────────────────────────┘
```

### Node-by-node explanation

#### 1. Application Form
A form trigger collects the candidate data and files.

Required fields include:

- Job openings PDF
- Resume PDF
- Mode
- Candidate email
- Full name
- Experience in years
- Email subject
- Email body

Optional fields:

- Phone number

This is the entry point for the workflow and is designed to be easy to use in production or testing.

#### 2. Read Resume
The resume PDF is parsed using the Extract from File node.

The extracted text is normalized and converted to lowercase so it can be matched with the JD text.

#### 3. Read Job Descriptions PDF
The JD PDF is also extracted and parsed from a PDF document.

The workflow expects JD sections to be separated in a consistent format so each job block can be detected correctly.

#### 4. Match Jobs and Prepare Emails
This is the main decision-making node.

It performs a keyword-based match between:

- the resume text
- the PDF of job descriptions
- the candidate profile such as years of experience

It applies:

- experience fit checks
- hard blocker logic
- soft blocker logic
- skill overlap checks
- recruiter deduplication

If the job passes the evaluation, it prepares the email payload for the recruiter.

#### 5. Send Application Email
The final node sends the generated email via Gmail.

The resume is attached as the uploaded PDF and the candidate name is used as the sender name.

---

## What the Workflow Checks

### Experience fit
The workflow compares the candidate's experience with the required years from the JD.

Logic:

- allows a tolerance of approximately ±1 year
- rejects jobs outside the allowed range

### Hard blockers
If the JD mentions technologies or stacks that are considered incompatible with the candidate profile, the job is rejected.

Examples from the workflow include:

- Salesforce
- SAP
- Finacle
- ETL / Data Warehouse
- Bluetooth
- Wi-Fi / 5G / Telecom
- AR / VR
- LoadRunner / NeoLoad
- TOSCA
- Mainframe
- Kafka

### Soft blockers
These are rejected only if the job also does not mention the candidate's automation tool stack, especially Playwright or Cypress.

Examples include:

- Java
- C#
- Selenium
- Python
- JMeter
- Appium

### Skill matching
The workflow checks whether the JD and resume share at least 3 qualifying skills.

Examples of tracked skills:

- Playwright
- Cypress
- API testing
- Manual testing
- Functional testing
- Regression testing
- Automation
- AI / LLM testing
- JavaScript / TypeScript
- CI/CD
- Agile / Scrum
- Jira / Azure DevOps
- QA leadership
- Defect management

This makes the workflow especially well suited for QA, test automation, software testing, and AI-enabled QA roles.

---

## Input and Output Behavior

### Inputs
The workflow accepts:

- JD PDF with multiple job postings
- resume PDF
- candidate information
- email subject and email body
- mode selection

### Outputs
The workflow creates:

- recruiter-specific email payloads
- subject lines
- body text
- file attachments
- Gmail sends to either preview inbox or actual recruiters

In Preview mode, the app sends everything to the candidate's own email so the candidate can validate before going live.

---

## Configuration Details

### Email provider
The workflow uses Gmail and the Gmail node with Gmail OAuth2 credentials.

The email is sent from the configured Gmail account, and the sender name is set to the candidate's full name.

### n8n version
This workflow is designed for recent n8n versions and uses the following node types:

- Form Trigger v2.6
- Extract from File v1.1
- Code v2
- Gmail v2.2

No strict version lock is required, but a recent n8n instance is recommended.

### Environment variables / secrets
No custom environment variables are required for the core workflow.

The only real setup needed is:

- connect a Gmail OAuth2 credential in n8n
- create the workflow form
- upload the JD and resume PDFs

---

## Form Setup

The workflow includes a form with a set of fields like:

- Job openings PDF
- Resume PDF
- Mode
- Candidate email
- Full name
- Phone number
- Experience years
- Email subject
- Email body

### Preview vs Live mode

#### Preview mode
- sends all emails to the candidate's own email
- adds a `[PREVIEW to recruiter@...]` prefix to the subject
- allows safe testing before actual recruiter outreach

#### Live mode
- sends emails to the recruiter addresses extracted from the JD PDF
- uses the same body and subject for all matched roles

---

## Matching Logic Details

The matching is rule-based and not AI-powered.

This means it is easy to understand and edit for different industries or profiles.

### Rules used by the workflow

- Candidate experience must fall within a tolerance band relative to job requirements.
- Resume and JD must share at least 3 matching skills.
- Hard blockers are used to reject roles with incompatible technologies.
- Soft blockers are used as a secondary filter.
- Recruiter emails are deduplicated.
- A recruiter is emailed only once per run.

### Limitations
The workflow is intentionally practical but has some limits:

- It expects a structured JD PDF with job blocks separated clearly.
- It relies on text extraction, so scanned or image-only PDFs may not work.
- It does not use OCR.
- It does not tailor the email body per job or recruiter.
- It does not persist previous run results.
- It does not log rejections in a dedicated database.

---

## Execution Behavior and Performance

### Typical run timing
Based on recent runs:

- each successful run took about 4 seconds for the main flow
- each additional matched email adds roughly 1 second
- a run with 20–30 matched jobs usually takes around 20–40 seconds

### Email volume
The workflow has no hard cap on matched jobs but the practical limit depends on the email provider.

For Gmail:

- personal Gmail account: roughly 500 emails/day
- Google Workspace: roughly 2,000 emails/day

### Example realistic behavior
- If no job fits, the run still succeeds but no email is sent.
- If matches exist, the workflow sends one email per unique recruiter address.
- Duplicate recruiter emails are ignored.

---

## Security and Data Handling

This workflow:

- reads candidate documents from the form upload
- extracts PDF text for matching
- sends the resume as an attachment
- uses your configured Gmail account for sending

For production usage, it is recommended to:

- use a dedicated Gmail account or business account
- validate PDFs before sending
- review the job description input format before production use
- verify recruiter email addresses before performing live outreach

---

## Example Use Case

A QA engineer uploads:

- a PDF containing 30 job listings
- a resume highlighting Playwright, API testing, Cypress, Agile, CI/CD, and QA leadership

The workflow:

- extracts both documents,
- identifies the jobs that share at least 3 skills,
- rejects roles requiring incompatible stacks,
- and sends tailored email outreach to recruiters for the qualifying roles.

---

## Recommended Enhancements

This workflow is already useful as a practical outreach automation tool. Potential future improvements include:

- adding OpenAI / LLM-based job matching,
- adding a CSV export of matched jobs,
- adding recruiter response tracking,
- adding automatic follow-up emails,
- adding a pre-check to validate PDF layout before sending,
- adding email templates per role category,
- adding a rejection summary log for debugging and optimization.

---

## Best Practices

To make this workflow more reliable in production:

1. Use clean, structured JD PDFs.
2. Validate the resume text before sending.
3. Keep a dedicated preview workflow before using Live mode.
4. Review recruiter email extraction logs regularly.
5. Use a business Gmail account for larger sending volumes.
6. Keep the keyword lists updated to the target job domain.

---

## Summary

This n8n workflow is a smart recruitment automation pipeline that:

- collects candidate and role data,
- parses PDFs,
- matches skills and experience,
- filters unsuitable roles,
- and sends bulk recruiter emails with resume attachments.

It is particularly effective for QA and automation job seekers and is a great example of how n8n can automate a real-world hiring workflow in a structured and scalable way.

---

## Project Status

This project is a working n8n workflow demonstrating one practical use case for browser-based automation, PDF parsing, business logic, and Gmail outreach.

It is ideal for:

- portfolio showcase projects,
- technical demos,
- automation workflow presentations,
- recruitment automation experiments,
- and learning more about n8n workflow orchestration.

---

## License

This project is currently shared as a workflow demonstrator and can be adapted for personal or portfolio use.

If you want to publish it publicly, you may also add a dedicated license file depending on how you want the workflow to be reused.

---

## Contact / Author

Project repository:

- https://github.com/Abhishek-Githu-home/n8n-Project

This workflow was created as a personal automation project to streamline mass job applications using n8n.
