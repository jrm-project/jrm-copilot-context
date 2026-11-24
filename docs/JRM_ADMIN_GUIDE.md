# JRM Administration Guide

**Purpose:** Operator guide for the JRM Administration interface
**Scope:** Current administrator-visible functions verified from JRM source and templates on 24 November 2026
**Source of truth:** `jrm-project/jrm`

> This guide explains how to operate the JRM Admin interface. It documents verified behaviour from the current application implementation. If the interface or implementation differs from this guide, stop and verify the deployed version before acting.

## 1. Access and navigation

JRM Admin requires an authenticated administrator session. Unauthenticated access to protected administration routes redirects to the login page.

The login endpoint is rate limited to **5 attempts per minute and 20 attempts per hour**.

After login, the persistent Admin sidebar provides these primary areas:

1. Dashboard
2. Bookings
3. Parts
4. Catalogue
5. Research
6. Enrichment
7. Content
8. Gallery
9. Email
10. Security

The top bar also provides:

- **View website**: opens the public website in a new browser tab.
- **Log out**: clears the JRM application session and returns to the public site.

Some specialist pages, including catalogue staging, research agents, automatic enrichment and enrichment results, are reached from related workflows rather than appearing as primary sidebar entries.

---

## 2. Dashboard

### Purpose

The Dashboard is the operational overview for work requiring administrator attention.

### What it shows

- **New bookings**: bookings with status `new`.
- **Upcoming**: bookings with a preferred date today or later that are not completed or cancelled.
- **Parts action**: booking-part records whose status is `required` or `ordered`.
- **Research review**: research submissions with status `pending` or `conflict`.
- **Enrichment failures or retries**: enrichment batches in `retry_wait` or `failed_permanent`.
- **Recent bookings**: the eight most recently created bookings.

### Quick actions

The Dashboard provides shortcuts to:

- Edit website content.
- Manage the gallery.
- Add a motorcycle variant.
- Check the email system.

### Operating guidance

Use the Dashboard as the first operational check after signing in. The **Attention required** section is particularly useful for finding research reviews, enrichment failures and booking parts requiring action.

---

## 3. Bookings

### 3.1 Booking list

The Bookings page lists service requests with the newest first.

For each booking it displays:

- public booking reference;
- creation time;
- customer name and phone number;
- motorcycle year, manually entered make and model;
- preferred service date;
- booking status.

Select the booking reference to open the Booking Detail page.

### 3.2 Booking Detail

The Booking Detail page is the main operational workspace for an individual service booking.

It contains:

- current booking status;
- customer contact and service-location information;
- customer-entered motorcycle details;
- catalogue linkage;
- catalogue-assisted preparation;
- requested work and dates;
- expected service checklist;
- booking-part preparation;
- internal notes;
- research tasks;
- status history.

### 3.3 Update booking status

1. Open the booking.
2. Select the required value from **Booking status**.
3. Select **UPDATE STATUS**.

When the status actually changes, JRM:

- updates the booking status and modification time;
- writes an entry to `booking_status_history` with the old and new status;
- records the change as made by `admin`;
- calls the booking status notification process.

**Important:** A status change can trigger an external customer notification. Treat status changes as customer-facing actions, not merely internal labels.

Selecting the existing status again does not create another status-history event or send another notification.

### 3.4 Customer and motorcycle information

Booking Detail displays the customer name, phone, email, address and service address where supplied.

The motorcycle section displays the customer-entered year, make, model, variant, engine size, registration, VIN and odometer where supplied.

These customer-entered details are separate from the controlled motorcycle catalogue until the booking is linked to a catalogue variant.

### 3.5 Link a booking to a catalogue variant

Use **Catalogue link** when the customer's motorcycle can be matched to a controlled catalogue variant.

1. Review the customer-entered motorcycle details.
2. Choose the correct catalogue variant.
3. Select **LINK VARIANT**.

JRM updates the customer's motorcycle record with the selected `vehicle_variant_id`. It also attaches that variant to matching booking research tasks that were created without a variant.

If the required variant does not exist, use **Create variant** and establish the catalogue record first.

**Do not link a booking to an approximate or unverified variant merely to make preparation work.** Specifications and fitments belonging to the linked variant can subsequently be reused during preparation.

### 3.6 Catalogue-assisted preparation

Catalogue-assisted preparation combines:

- the linked motorcycle variant;
- its recorded specifications;
- service-template requirements;
- known part fitments.

It does **not** order parts.

A booking must be linked to a catalogue variant before preparation can run.

Select **REFRESH PREPARATION** to recalculate missing booking preparation items.

For each service requirement with a part category:

- if exactly one trusted fitment exists and there is not already a booking-part record for that category, JRM creates a booking part containing that known part;
- if there is not exactly one trusted fitment, JRM creates a category-only booking item requiring verification;
- if there are no trusted fitments, JRM creates a pending `part_fitment` research task unless an equivalent open task already exists.

Existing booking-part categories are not duplicated by refresh.

The completion message reports the number of booking items and research tasks created.

### 3.7 Create research for a missing preparation category

Where a preparation requirement has no known compatible part, select **CREATE RESEARCH TASK** for that category.

JRM creates a `part_fitment` research task with priority **70**, tied to the linked catalogue variant.

If an equivalent pending or in-progress research task already exists, another task is not created.

### 3.8 Expected service checklist

The Expected service checklist is generated from the service type's configured requirements. It can contain replacement items, inspections/checks and quantities.

If no requirements are configured for the service type, the page explicitly reports that condition.

### 3.9 Booking parts

Booking parts are the parts or consumables being prepared for a specific booking.

#### Add a booking part

1. Choose a **Category**, a **Specific part**, or both.
2. Enter a quantity from **1 to 100**.
3. Choose a status.
4. Add optional notes.
5. Select **ADD TO BOOKING**.

At least a category or a specific part is required.

If an invalid status is submitted when adding a part, JRM falls back to `suggested`.

A category-only record is valid and is useful where the specific part still needs research.

#### Update a booking part

For an existing booking part, update:

- quantity, from 1 to 100;
- status;
- notes.

Select **SAVE** to persist the change.

#### Remove a booking part

Select **REMOVE** to delete that booking-part record from the booking.

This removes the booking association. It does not delete the underlying known part from the Parts catalogue.

### 3.10 Internal notes

Internal notes are explicitly marked as **not visible to the customer**.

1. Enter the note, up to 4,000 characters.
2. Select **ADD INTERNAL NOTE**.

JRM stores the note against the booking and records the creator as `admin`.

### 3.11 Manual research tasks

A booking can create research tasks for:

- Part fitment
- Vehicle specification
- Service information
- Other

To create a task:

1. Choose the research type.
2. Set priority from **1 to 100**. The form defaults to **50**.
3. Describe precisely what must be researched, up to 1,000 characters.
4. Select **CREATE RESEARCH TASK**.

JRM prefixes the stored description with the booking ID. If the booking is already linked to a catalogue variant, that variant is attached to the research task.

### 3.12 Status history

Status History provides a chronological audit trail of booking status changes, including the new status, timestamp and recorded changer.

---

## 4. Parts

### Purpose

The Parts page maintains known parts which can later be assigned to bookings and linked to motorcycle variants as fitments.

### Add a part

1. Choose a part category.
2. Enter the manufacturer if applicable.
3. Enter the part number.
4. Enter an optional description.
5. Select **ADD PART**.

Category and part number are required.

The combination of manufacturer and part number must be unique. A duplicate is rejected rather than creating another part record.

### Known parts

The list shows:

- category;
- manufacturer, or `Generic / OEM` where blank;
- part number;
- description.

Creating a part here does not establish that the part fits any particular motorcycle. Fitment belongs in the relevant catalogue variant.

---

## 5. Motorcycle Catalogue

### Purpose

The controlled Motorcycle Catalogue stores makes, models, variants, verified specifications and part fitments.

This is trusted operational data. Administrators should not enter guessed values merely to complete a record.

### 5.1 Add a manufacturer

Enter the manufacturer name and select **ADD MAKE**.

The manufacturer slug is generated from the name. Duplicate manufacturers are rejected.

### 5.2 Add a motorcycle variant

Select **ADD VARIANT**, then complete the applicable fields:

- Manufacturer, required
- Model, required
- Variant / series
- Engine capacity, 25 to 10,000 cc if supplied
- Year from, 1900 to 2100 if supplied
- Year to, 1900 to 2100 if supplied
- Engine code
- Model code
- Market, default `AU`
- Notes

If both years are supplied, **Year to cannot be earlier than Year from**.

If the model does not already exist beneath the selected manufacturer, JRM creates it automatically.

A manually created variant is saved with verification status `manually_verified`.

### 5.3 Variant details

The variant detail page displays:

- make, model and variant;
- verification status;
- year range;
- engine capacity;
- engine and model codes;
- market;
- notes;
- specifications;
- part fitments.

### 5.4 Add or update a specification

Enter:

- Specification key
- Value
- Unit, optional
- Source / evidence note, optional

Select **SAVE SPECIFICATION**.

Spaces in the specification key are converted to underscores and the key is stored in lower case.

If the variant already has that specification key, JRM updates the existing record rather than creating another one.

Administrator-saved specifications are marked `manually_verified`.

**Operating guidance:** Record the source/evidence note whenever practical. Manual verification status should reflect an actual administrator verification step, not assumption.

### 5.5 Add a part fitment

Choose:

- a known part;
- optional position, such as front or rear;
- quantity from 1 to 100;
- verification status;
- optional notes.

Select **ADD FITMENT**.

If the chosen verification status is anything other than `unverified`, JRM records the verification time and `verified_by='admin'`.

The same part and position cannot be entered twice for a variant.

### 5.6 Update a fitment

An existing fitment can have its verification status and notes changed. Select **SAVE FITMENT** to save it.

Changing to a verified status records the current verification time and identifies the verifier as `admin`.

---

## 6. Free Catalogue Staging

### Purpose

Free Catalogue Staging imports open motorcycle make/model data from **VehiclesDB** into an untrusted staging area.

Staging data is **not** treated as verified year, variant or fitment data.

### 6.1 Download and refresh staging data

Select **DOWNLOAD AND REFRESH STAGING DATA**.

JRM:

- downloads the source CSV with a 60-second network timeout;
- refuses downloads over **25 MB**;
- validates the expected CSV schema;
- accepts rows whose `kind` is `motorcycle`;
- saves the downloaded CSV to `/var/lib/jrm/imports/vehiclesdb-vehicles.csv` with mode `0640`;
- updates staging records and source metadata.

If no motorcycle records are found, the import is refused.

### 6.2 Browse staging

The staging browser supports:

- free-text make/model search;
- Australia;
- New Zealand;
- Australia + New Zealand;
- all motorcycles.

It displays source make/model, AU/NZ availability and global popularity decile.

### 6.3 Promote one model

Select **PROMOTE** beside a staging record.

JRM creates or reuses the matching manufacturer and model in the controlled catalogue, then marks the staging record as promoted.

**Promotion does not create years, engine capacities or variants.** Those require separate evidence and controlled variant creation.

### 6.4 Promote common Australian models

Choose a maximum global popularity decile and select **PROMOTE COMMON AU MODELS**.

JRM promotes Australian-available, not-yet-promoted models with a global popularity decile at or above the selected popularity threshold. The operation is capped at **1,000 staging records per run**.

The UI provides decile choices 1 through 5, with 1 through 3 selected by default. The backend constrains submitted values to 1 through 10.

Again, promotion only creates manufacturer/model records. It does not fabricate variants or specifications.

---

## 7. Research

### Purpose

Research separates untrusted candidate information from trusted catalogue data.

Research agents can work on tasks and submit candidate claims with evidence. Administrator review controls whether eligible claims enter the trusted catalogue.

### 7.1 Research overview

The Research screen shows:

- AI research submissions;
- submission ID, agent, type, motorcycle and status;
- the research task queue, including type, description, priority and status.

### 7.2 Review a submission

Open a submission by its ID.

Review:

- submitting agent;
- submission type;
- target motorcycle;
- proposed claim;
- evidence summaries;
- cited source links;
- any detected conflict with existing trusted data.

If the submission is `pending` or `conflict`, the administrator can enter a review note/rejection reason and choose:

- **APPROVE AND PROMOTE**
- **REJECT**

### 7.3 Reject a research submission

Rejecting the submission:

- marks it `rejected`;
- records the review timestamp and rejection reason;
- writes an admin rejection event to the audit log.

If no reason is entered, JRM stores `Rejected by administrator.`

### 7.4 Approve and Promote

**This is a consequential action.** Unlike enrichment's **Mark Useful**, research **Approve and Promote** can modify trusted catalogue data.

For a `vehicle_specification` submission, approval can insert or update a specification and marks the resulting trusted value as `supplier_verified`, with a source note referencing the approved research submission.

For a `part_fitment` submission, approval can insert or update a fitment and marks it `supplier_verified`, including verification time and `verified_by='approved research'`.

If the candidate conflicts with an existing trusted value:

- the conflict is recorded;
- the submission remains/returns `conflict`;
- existing trusted data is **not overwritten**;
- the administrator is returned to the submission for further review.

After successful approval:

- the submission becomes `approved`;
- an associated research task, where supplied, becomes `completed`;
- an approval event is written to the audit log.

**Before selecting APPROVE AND PROMOTE, review both the claim and its evidence.**

### 7.5 Research agents

The Research Agents page supports creation and enable/disable control of agents.

To create an agent, supply:

- Name, required
- Provider, optional
- Model, optional

JRM generates an authentication token beginning with `jrm_`.

**The generated token is displayed only once.** JRM stores a cryptographic hash rather than the plaintext token. Copy and store the token securely at creation time.

Configured agents display name, provider/model, last-used time and enabled state.

Use **ENABLE** or **DISABLE** to toggle whether the agent is enabled.

---

## 8. Enrichment

### Purpose

Enrichment generates and processes research intended to improve motorcycle catalogue coverage without automatically trusting AI output.

There are two related layers:

1. the enrichment task queue;
2. automatic enrichment batches/results.

### 8.1 Generate or refresh enrichment tasks

Select **GENERATE / REFRESH ENRICHMENT TASKS**.

For promoted motorcycle models, JRM generates four deduplicated research streams covering:

- generations;
- engine variants;
- service specifications;
- common service parts.

Task priority is weighted by global popularity decile. Existing equivalent tasks are not duplicated.

### 8.2 Cancel an enrichment task

Pending or in-progress enrichment tasks can be cancelled from the queue.

Cancellation changes the task status to `cancelled`.

### 8.3 Automatic enrichment dashboard

Automatic Enrichment displays:

- configured model;
- local daily budget and usage;
- worker enabled state;
- timer state;
- counts for pending, processing, retry wait, completed and permanent failures;
- the next eligible batch;
- recent results;
- retry/permanent failures.

The page explicitly treats provider research as **untrusted until reviewed**.

### 8.4 Run one eligible batch

Select **RUN ONE ELIGIBLE BATCH** to invoke the configured worker immediately for one live eligible item.

The application runs the worker with `--live-one`, captures its output and allows up to **240 seconds** for the process.

The resulting worker output, or error, is returned as an Admin flash message.

### 8.5 Enable or pause the automatic timer

- **ENABLE AUTOMATIC TIMER** runs `systemctl enable --now jrm-enrichment-worker.timer`.
- **PAUSE AUTOMATIC TIMER** runs `systemctl disable --now jrm-enrichment-worker.timer`.

This directly changes the automatic enrichment systemd timer on the host. Use these controls deliberately.

### 8.6 Retry a failed batch

**RETRY** is available only for batches in `retry_wait` or `failed_permanent`.

Retry:

- returns the batch to `pending`;
- clears its next-attempt time and recorded error details;
- returns cancelled tasks connected to that batch to `pending`.

### 8.7 Cancel an automatic batch

**CANCEL** can cancel a batch whose status is `pending`, `retry_wait` or `failed_permanent`.

It also cancels associated model enrichment tasks that are still `pending` or `in_progress`.

### 8.8 Review enrichment results

Enrichment results can show:

- candidate findings;
- applicability;
- confidence supplied by the provider;
- identified gaps;
- grounding/cited sources;
- parsed candidate sections for automatic batches.

Available review decisions are:

- **MARK USEFUL**
- **REJECT**

**Critical distinction:** Marking an enrichment result useful does **not** promote claims into the trusted catalogue. It records the review decision only. Trusted variants, specifications, parts and fitments must subsequently be created or updated through a controlled/manual process or an approved research submission.

---

## 9. Website Content

### Purpose

The Content page edits public website wording. Navigation destinations remain application-controlled.

### Editable content

#### Business details

- Business name
- Telephone
- Email address

#### Navigation labels

- Services
- Mobile service
- Gallery
- About
- Contact
- Call prefix

Only labels are editable. Page anchors/destinations remain fixed by the application.

#### Hero

- Eyebrow
- Heading
- Text

#### Mobile service and About

- Mobile section heading
- Mobile section text
- About heading
- About text

#### Footer

- Description
- Location
- Copyright name

### Save behaviour

Select **Save website content** to write all listed fields into the application `settings` table.

**The UI explicitly warns that these changes affect the production website.** Review the wording before saving.

Use **Preview website** or the top-bar **View website** link to inspect the public site.

---

## 10. Gallery

### Purpose

The Gallery page manages public recent-work photographs.

### 10.1 Upload a photograph

Supported image formats:

- JPEG
- PNG
- WEBP

Requirements:

- image must be at least **320 x 240 pixels**;
- the Admin UI sets a maximum upload of **10 MB**;
- the application rejects invalid/unsupported image data and excessive pixel dimensions.

Enter an optional caption up to **200 characters**, then choose **Validate and upload**.

JRM verifies the image, reloads it through the image library, converts/resaves it into a clean supported representation, assigns a UUID-based filename and saves it with file mode `0640`.

JPEG images are resaved at quality 88 with optimisation and progressive encoding. PNG is optimised. WEBP is resaved at quality 88.

### 10.2 Delete a photograph

Select **Delete** beside a gallery image and confirm the deletion prompt.

Deletion removes:

- the uploaded image file, if present;
- the image's database record.

This is destructive and there is no Admin undelete function shown in the current implementation.

---

## 11. Email

### Purpose

The Email page reports whether SMTP notification configuration is complete and provides a controlled test function.

### Configuration status

The page displays:

- enabled state;
- SMTP host;
- port;
- username;
- password state as only `configured` or `not configured`;
- From address;
- Admin recipient;
- STARTTLS state;
- SSL state.

The SMTP password itself is **not displayed**.

If configuration problems exist, they are listed at the top of the page.

### Send a test email

Select **SEND TEST EMAIL**.

If configuration validation reports problems, JRM refuses to send and displays the issues.

If ready, JRM sends an `admin_test` message to the configured administrator address with subject:

`JRM email configuration test`

Recent email activity lists up to the latest **100** email log entries, including time, message type, recipient, status and error detail.

---

## 12. Security

### Purpose

The Security page changes the JRM application password. Cloudflare Access remains the first authentication layer where configured in the deployed environment.

### Password requirements

A new JRM password must:

- contain at least **14 characters**;
- contain both uppercase and lowercase letters;
- contain at least one number;
- differ from the current password.

### Change the password

1. Enter the current password.
2. Enter the new password.
3. Confirm the new password.
4. Select **Change password**.

Password-change attempts are rate limited to **5 per hour**.

After a successful password change:

- the stored password hash is replaced;
- the current JRM session is cleared;
- the administrator is redirected to login and must sign in again.

---

## 13. Operational safeguards

### Trusted versus untrusted data

JRM deliberately separates candidate research from trusted catalogue data.

- Free catalogue staging is untrusted make/model source data.
- Enrichment output is untrusted research until reviewed.
- **MARK USEFUL** does not modify trusted catalogue content.
- **APPROVE AND PROMOTE** on an eligible research submission can modify trusted catalogue specifications or fitments.
- Manually entered variants/specifications/fitments should only be marked through the Admin workflow when the administrator has sufficient evidence.

### Customer-facing actions

Treat the following as potentially externally visible:

- booking status changes, because notifications can be sent;
- website content changes;
- gallery uploads and deletions;
- email test messages.

### System-control actions

Treat these as operational controls rather than ordinary content edits:

- enabling or pausing `jrm-enrichment-worker.timer`;
- running one live enrichment batch;
- retrying or cancelling enrichment batches;
- changing the JRM password.

---

## 14. Troubleshooting quick reference

### Cannot access Admin

- Confirm the JRM application itself is reachable.
- Confirm any outer access layer, including Cloudflare Access where configured, has succeeded.
- Confirm valid JRM administrator credentials.
- Note that login attempts are rate limited.

### Booking preparation says the booking is not linked

Link the booking to the correct controlled catalogue variant before using **REFRESH PREPARATION**.

### Preparation creates a research task instead of choosing a part

This means JRM did not find exactly one trusted fitment suitable for automatic booking-part selection. Review the variant's fitments and use the Research workflow rather than guessing.

### Research approval reports a conflict

Existing trusted data has not been overwritten. Review the existing value and candidate evidence. Resolve the discrepancy deliberately before changing trusted catalogue data.

### Enrichment result looks correct but nothing appears in the trusted catalogue

That is expected. **MARK USEFUL** records review status only. Use the controlled catalogue workflow or an approved research submission to change trusted data.

### Email test is refused

Review the problems listed on the Email page. The application will not attempt the test while `email_status()` reports configuration problems.

### Gallery upload is rejected

Confirm the file is a real JPEG, PNG or WEBP image, is at least 320 x 240 pixels, and meets the upload/application safety limits.

### Password changed successfully but Admin returned to login

That is expected. A successful password change clears the current session and requires a fresh login.

---

## 15. Documentation maintenance rule

This guide is part of the JRM definition of done for administrator-facing functionality.

Whenever a change modifies an Admin screen, field, button, workflow, validation rule, side effect, warning or operational procedure:

1. review `docs/JRM_ADMIN_GUIDE.md` in the same feature branch;
2. update the relevant section before the tested release is promoted to Production;
3. verify the guide against the implemented templates and backend behaviour;
4. publish the current guide to the public Copilot context repository after the authoritative private documentation is committed.

Do not document proposed functionality as though it is already implemented.
