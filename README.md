# GHL Appointment Automation

## 1. Architecture Overview
An automated workflow built using n8n that processes GHL appointment webhooks, normalizes the data, stores it in a mock CRM (Airtable), and sends formatted Slack notifications
<img width="1800" height="1126" alt="Screenshot 2025-12-07 at 7 45 56 PM" src="https://github.com/user-attachments/assets/5d592fc4-e8c3-40a7-a466-cd89227b5211" />

Flow:
- GHL n8n Webhook 
- Normalize/Transform 
- Upsert Contact
- Create Appointment
- Slack Notification
- Callback Save

## 2. Tools Used
1. n8n – Main automation engine
2. Webhook Receiver – Entry point for GHL appointment event
3. Slack Incoming Webhook – Sends formatted notifications
4. Airtable API (Mock) – Simulated CRM for contacts & appointments

## 3. Workflow Explanation
GHL sends a POST request to n8n when a new appointment is booked.

Step 1 – Receive Appointment Webhook
GHL sends a POST request to n8n when a new appointment is booked.
Payload includes:
- Customer name
- Start & end times
- Appointment status
- Lead source
- Project type

Step 2 – Validate & Normalize
The workflow cleans and structures the raw data:
- Combines first + last name
- Converts timestamps to ISO format
- Ensures fallback values if fields are missing
- Maps GHL field names internal naming convention
- This ensures all downstream nodes receive clean and predictable data.

Step 3 – Create Contact (Airtable Mock)
Customer record is created in the mock CRM:
- Name
- Phone or Email
- Lead source

Step 4 – Create Appointment (Airtable Mock)
The appointment is linked to the created contact:
- Start / End time
- Status
- Project type

Step 5 – Send Slack Notification
A message is posted to Slack using a webhook URL (secured via template variable in JSON).
New Appointment Booked!
Name: Jane Doe
Start: 2025-01-05 15:00
End: 2025-01-05 16:00
Status: booked
Project Type: Consultation
Lead Source: Facebook Ads

<img width="482" height="201" alt="Screenshot 2025-12-07 at 7 51 57 PM" src="https://github.com/user-attachments/assets/232a3452-59aa-4100-b0a2-918bf7c0336d" />


## 4. Setup Instructions
1. How to run
   - Import the workflow JSON file into n8n.
   - Replace environment variables (e.g., SLACK_WEBHOOK_URL).
   - Start the workflow and activate.
2. How to test via Postman
   - POST to your n8n webhook URL: https://your-n8n-domain/webhook/test-appointment
   - Example payload
     {
        "event": "appointment.created",
        "appointment_id": "apt_12345",
        "location_id": "loc_abc",
        "calendar_id": "cal_01",
        "contact": {
          "id": "con_999",
          "first_name": "Jane",
          "last_name": "Doe",
          "email": "jane@example.com",
          "phone": "+1 555 222 3333"
        },
        "appointment": {
          "starts_at": "2025-01-05T15:00:00Z",
          "ends_at": "2025-01-05T16:00:00Z",
          "status": "booked",
          "custom_fields": {
            "project_type": "solar",
            "lead_source": "Facebook Ads"
          }
        }
      }
     - How to trigger
       - Run via GHL webhook

## 5. Files Included
- /workflow/ghl-appointment-automation.json
