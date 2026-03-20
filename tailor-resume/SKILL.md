---
name: tailor-resume
description: Tailor Babajide's resume for a specific job application. Fetches the job posting, reviews the current resume, and produces a fully updated version optimised for the role. Use when the user wants to apply for a job and needs a tailored resume.
argument-hint: <job posting URL or paste job description>
---

Tailor Babajide's resume for a specific job application.

## Master Resume

The master resume is stored at:
```
/Users/babajideokusanya/Desktop/Resumes/Babajide_Okusanya_Resume_Glean.md
```

This is the most up-to-date version and should be used as the base for all tailoring. Always read it fresh at the start of each session.

## Workflow

### Step 1 — Load the job posting
- If the user provides a URL, fetch the full job description using WebFetch
- If the user pastes the JD directly, use that
- Extract: job title, company, key responsibilities, required qualifications, preferred qualifications, and any language/keywords that appear repeatedly

### Step 2 — Read the master resume
- Read the master resume file in full
- Note the current summary, skills, and all experience bullets

### Step 3 — Gap analysis
Run a silent gap analysis before writing anything. Identify:
- **Strong matches** — experience that maps directly to the JD
- **Weak matches** — experience that is relevant but not framed correctly
- **Missing signals** — things the JD asks for that aren't surfaced anywhere on the resume
- **Irrelevant content** — sections or bullets that add no value for this specific role

### Step 4 — Check Granola for additional material
Use `query_granola_meetings` to search for relevant past work, projects, or interview discussions that might surface additional bullets or talking points not currently on the resume.

Query examples:
- Search for the company name or role type
- Search for relevant skills or domains mentioned in the JD
- Search for "interview" to find past interviews where Babajide described work in detail

### Step 5 — Draft the tailored resume
Produce a fully rewritten resume with:

**Summary:** 3 sentences max. Lead with the role type and core value proposition. Use language from the JD. No em dashes.

**Skills:** Reorder categories so the most relevant to the role appears first. Remove skills that have no relevance to this role. Add any relevant skills that are on the master resume but not listed.

**Experience:**
- Rewrite bullets to mirror JD language where the underlying work is genuinely the same
- Lead every bullet with the customer/outcome, not the technology
- Preserve all real metrics — never invent or inflate numbers
- Trim or remove bullets that add no signal for this role
- Add bullets from Granola research if they are genuinely relevant and accurate

**Format rules:**
- No em dashes anywhere
- Use pipe separators for dates and education
- Bullet points for all experience
- Max 2 pages
- Save as markdown

### Step 6 — Show the analysis first
Before showing the full resume, present a brief summary:
- What changed and why
- Any gaps that couldn't be filled from existing experience
- Any flags (location requirements, missing qualifications, etc.)

Ask the user if they want to proceed or make adjustments before showing the full tailored resume.

### Step 7 — Show the full tailored resume
Present the complete resume as clean copy-paste text.

Ask: "Want me to save this as a markdown file to your Desktop?"

### Step 8 — Save the file
If the user confirms, save to:
```
/Users/babajideokusanya/Desktop/Resumes/Babajide_Okusanya_Resume_[Company].md
```

## Important Rules

- Never invent metrics or experiences — only use what is real and verifiable
- Never make Babajide sound like a different person — keep the voice consistent
- If a role is a stretch or has a meaningful gap, flag it honestly rather than papering over it
- Always check Granola — past interviews often contain richer descriptions of work than the resume
- The master resume file should never be overwritten — always save tailored versions separately
