# Scenario Breakdown

This breakdown is based only on the visible contents of the Make.com blueprint JSON files.

## Scenario 1: Missed Call To Tally Form

Blueprint:

`scenarios/missed-call-to-tally-form.blueprint.json`

Visible scenario name:

`Example Home Services | Missed Call -> Lead Form`

### Modules

1. **Custom Webhook**
   - Module: `gateway:CustomWebHook`
   - Designer label: `Recieve call`
   - Receives call-related fields such as `From`, `To`, `CallSid`, `CallStatus`, and location/country fields.

2. **Datastore Lookup**
   - Module: `datastore:GetRecord`
   - Designer label: `check if number has been called`
   - Looks up a datastore record using the caller phone number: `{{1.From}}`.

3. **Router**
   - Module: `builtin:BasicRouter`
   - Splits the flow based on whether the caller has a recent stored timestamp.

4. **Send SMS**
   - Module: `twilio:SendSMS`
   - Designer label: `send out lead form`
   - Sends an SMS to `{{1.From}}`.
   - The SMS body includes a Tally form link and a WhatsApp fallback link.
   - Runs when no previous timestamp exists, or when the previous timestamp is older than 24 hours.

5. **Add/Update Datastore Record**
   - Module: `datastore:AddRecord`
   - Designer label: `add number (or add date called)`
   - Stores the caller number as the key and writes the current timestamp to `Missed_call_sms_guard`.
   - Uses overwrite mode.

6. **Webhook Response**
   - Module: `gateway:WebhookRespond`
   - Designer label: `hangup`
   - Returns a TwiML-style XML response containing `<Hangup/>`.

### Routing Logic

The SMS route runs if:

- `Missed_call_sms_guard` does not exist, or
- `Missed_call_sms_guard` is older than `addHours(now; -24)`.

The non-SMS route runs if:

- `Missed_call_sms_guard` exists, and
- `Missed_call_sms_guard` is newer than `addHours(now; -24)`.

In plain English: the automation sends the Tally link only if the caller has not already been recorded in the last 24 hours.

## Scenario 2: Lead Summary Email

Blueprint:

`scenarios/lead-summary-email.blueprint.json`

Visible scenario name:

`Example Home Services | Lead Summary`

### Modules

1. **Tally New Response Trigger**
   - Module: `tally:watchNewResponse`
   - Designer label: `GET LEAD (form)`
   - Receives form fields including name, phone number, email, postcode, address, homeowner status, job description, property type, current wall/surface, timing, known issues, photos, and extra notes.

2. **OpenAI Response Module**
   - Module: `openai-gpt-3:createModelResponse`
   - Designer label: `Analyse LEAD`
   - Uses the form submission to generate a short HTML lead card.
   - The prompt asks for priority, contact details, job summary, timing, photos, missing information, and next action.

3. **Send Email**
   - Module: `google-email:sendAnEmail`
   - Designer label: `SEND SUMMARY`
   - Sends the generated HTML summary as a raw HTML email.

## Relationship Between The Scenarios

The first scenario pushes callers toward the Tally form by SMS. The second scenario starts when that Tally form is submitted and turns the submission into an email summary.
