# Appointment Booking Confirmation

An n8n booking flow for an HVAC business. A website form or other client sends the booking details to a webhook. The workflow validates the data, creates a one-hour Google Calendar event, emails the customer a confirmation, and logs every outcome to Google Sheets. Failed bookings trigger a Telegram alert.

## Architecture

```
Webhook (POST /book-appointment)
   → Valid Booking Data?
        ├── Yes → Create Calendar Event
        │          ├── Success → Send Confirmation Email
        │          │              ├── Success → Log Confirmed
        │          │              └── Error   → Log Failed Booking → Send Failure Alert (Telegram)
        │          └── Error   → Log Failed Booking → Send Failure Alert (Telegram)
        └── No  → Log Invalid Booking
```

## Features

* Validates the customer's name, email format, and service before anything is created
* Creates a one-hour calendar event with the customer's name, email, and service in the title and description
* Sends a confirmation email with the date and time of the appointment
* Logs every booking to Google Sheets: confirmed, invalid, or failed
* Sends a Telegram alert when a calendar event or confirmation email fails
* Errors in the calendar and email steps are caught and routed to the failure log, so the workflow does not stop silently

## Tech Stack

* n8n 2.x (workflow automation)
* Google Calendar API (appointment events)
* Gmail API (customer confirmations)
* Google Sheets (booking log)
* Telegram Bot API (failure alerts)

## Setup

1. Create a Google Calendar named `HVAC Appointments` and copy its calendar ID from the calendar settings.
2. Create a Google Sheet with a tab named `Sheet1` (or change the tab name in the workflow). Use these headers in row 1:

```
customer_name | customer_email | service | date | time | status | logged_at
```

3. Import `appointment_booking_confirmation.json` into n8n.
4. Add credentials: Google Calendar OAuth2, Gmail OAuth2, Google Sheets OAuth2, and Telegram. Assign them to the matching nodes.
5. Replace these placeholders:
   - `YOUR_CALENDAR_ID@group.calendar.google.com`: your calendar ID (Create Calendar Event node)
   - `YOUR_SHEET_ID`: your Google Sheet ID (all three Log nodes)
   - `YOUR_TELEGRAM_CHAT_ID`: your chat ID (Send Failure Alert node)
6. Activate the workflow and send a POST request to the webhook.

## Sample Request

```
{
  "customer_name": "Ali Raza",
  "customer_email": "ali.test@example.com",
  "service": "AC Tune-up",
  "date": "2026-11-05",
  "time": "10:00"
}
```

Expected result: a calendar event on 5 November 2026 from 10:00 to 11:00, a confirmation email to the customer, and a new row in the booking log.

## Project Notes

* Built as part of my AI Automation portfolio.
* The confirmation email is signed "CoolAir HVAC". Change this text in the Send Confirmation Email node to match your business.
* Dates and times are taken from the request as they are. Send them in `YYYY-MM-DD` and `HH:mm` format.
