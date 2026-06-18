# Data Flow

This data-flow description is based only on the visible blueprint JSON.

## Call To Form Flow

```mermaid
flowchart LR
  A["Call webhook"] --> B["Read caller number from From"]
  B --> C["Look up caller number in Make datastore"]
  C --> D{"Caller timestamp exists and is less than 24h old?"}
  D -->|"No"| E["Send SMS with Tally form link"]
  E --> F["Store caller number and current timestamp"]
  F --> G["Return XML hangup response"]
  D -->|"Yes"| H["Return XML hangup response"]
```

### Data Used

- Caller phone number: `From`
- Datastore key: caller phone number
- Datastore timestamp field: `Missed_call_sms_guard`
- SMS destination: caller phone number
- SMS content: lead-form link and WhatsApp fallback

## Form To Summary Flow

```mermaid
flowchart LR
  A["Tally form response"] --> B["Form fields mapped into AI prompt"]
  B --> C["OpenAI module creates HTML lead card"]
  C --> D["Gmail module sends summary email"]
```

### Form Fields Visible In The Blueprint

- Name
- Phone number
- Email
- Postcode
- Property address
- Homeowner status
- Work required
- Property type
- Current wall/surface
- Timing
- Known issues
- Photo uploads
- Extra notes

### AI Output Requested By The Prompt

- Lead priority
- Contact details
- Job summary
- Timing
- Photo status
- Missing information
- Suggested next action
