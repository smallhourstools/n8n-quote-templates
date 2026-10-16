# Small Hours: n8n templates for service businesses that quote by email

There is one person behind Small Hours. The work is done by a team of AI agents, who built the five free n8n workflows in this repository for small service businesses (movers, cleaners, contractors, installers) that quote jobs by email and keep track in Google Sheets. Each one was tested with real Gmail, Google Sheets and Google Calendar accounts before it was published here.

The five use the same Google Sheet (a tab named `Requests`), so each works on its own. The last two both read `closed_at`, so read their notes before you run them together.

| Folder | What it does | You need |
|---|---|---|
| `quote-requests/` | A quote request form that saves each request in Google Sheets and emails the customer: a confirmation, or a short list of what is missing | Google Sheets, Gmail |
| `booking/` | When you mark a quote `accepted`, the customer gets a personal link, picks a free time in your Google Calendar and gets a reminder the day before | Google Sheets, Google Calendar, Gmail |
| `weekly-summary/` | One email every Monday: open quotes and their value, quotes won and lost, the close rate, and the quotes open too long | Google Sheets, Gmail |
| `review-request/` | Once a job has a date in `closed_at`, the customer gets one email asking for a review, and never a second | Google Sheets, Gmail |
| `payment-reminder/` | Jobs with a date in `closed_at` that are not marked paid get a polite email, at most twice | Google Sheets, Gmail |

## How to use one

1. Download the `workflow.json` from its folder.
2. In n8n, create a new workflow and import the file (or paste its contents into the editor).
3. Read the yellow **About** note in the top left corner. It has the setup steps for that workflow, and each white note explains the nodes next to it.
4. Connect your own Google credentials. The files have no credentials, sheet IDs or live URLs in them.

## quote-requests

1. A customer fills the quote request form.
2. The workflow checks the details you need to quote and saves the request as `ready_to_quote` or `missing_info`.
3. You get an email with the request at `owner_email`.
4. The customer gets a confirmation, or a list of what is missing.

Tested: a complete request, an incomplete one, the same request sent again (not saved twice), two different requests one second apart (both saved), a phone number starting with `+`, and an empty `owner_email` (the customer still gets their email). Two identical submits in the same second can both get through; the canvas note says so.

Also in the n8n template library: https://n8n.io/workflows/20976

## booking

1. Set a quote's `status` to `accepted` in the sheet. Within 15 minutes the customer gets a personal booking link.
2. The customer sees only the free times in your calendar and books one. The job goes to Google Calendar and the row to `scheduled`.
3. Every morning, customers with a job tomorrow get a reminder.

Tested: a time that is already taken does not show; a time taken between opening the link and sending it is refused; a link with a wrong reference shows an error page; with two jobs booked, only tomorrow's gets the reminder.

Needs the form path `book-job` free in your n8n.

## weekly-summary

1. Every Monday at 8:00 the workflow reads the sheet and groups the rows by `status`.
2. You get one email with the totals.
3. Closed quotes get a date in `closed_at`, so each one is counted once.

The first run counts every quote already closed in the sheet. A test run also marks them as counted.

## review-request

1. Every day at 10:00 the workflow looks for rows with a date in `closed_at` and no review asked yet. You type that date by hand when a job is done.
2. The customer gets one email with your review link, in the thread of their quote when `thread_id` is filled, as a new email if not.
3. The row gets `review_asked` and the date in `reply_label`, and it is never asked again. The row is marked before the email is sent, so a customer is not asked twice.

Rows with status `lost` or `no_reply` are skipped, and so are rows whose `job_start` is still in the future. If Gmail refuses the email, the row says `review_failed` and you get an email; it is not retried.

Nothing is sent until you replace the default `business_name`, `owner_email` and `review_link` in **Settings**. Limitations: do not run it next to `weekly-summary/`, which also fills `closed_at`; and it asks for a review whether or not the customer has paid, so think before you run it with `payment-reminder/`.

## payment-reminder

1. Every day at 10:00 the workflow looks for rows with a date in `closed_at` (the day you finished or invoiced the job) that are not marked paid.
2. The first reminder goes out 3 days after that date and the second 7 days after the first. Then it stops. The days and the maximum are in **Settings**.
3. Each email goes in the quote thread when `thread_id` is filled, as a new email if not. The row gets `pay_reminder_1` (or 2) and the date in `reply_label`.

When the customer pays, type `paid` in `reply_label`. Rows with status `lost` or `no_reply` are skipped. If Gmail refuses the email, the row says `pay_failed` and you get an email; it is not retried.

Tested with seven rows: a job due for its first reminder, one not due yet, one due for its second, one that already had two, one marked `paid`, one with a thread that does not exist (sent as a new email) and one marked `lost`. A second run sent nothing again.

Nothing is sent until you replace the default `business_name`, `owner_email` and `payment_info` in **Settings**. Limitations: it does not know who paid, so mark `paid` before 10:00; and a `closed_at` that is not a date like 2026-10-14 is skipped without a message.

## Google connection

If your Google Cloud app is External and in Testing, the connection expires after 7 days; publish it to avoid this (Google then shows an "unverified app" warning you can click through). Google's setup guide for n8n: docs.n8n.io/integrations/builtin/credentials/google/

## Paid template

**Gmail quote follow-up for n8n that stops when the customer replies.** $19, one-time payment.

You send the quote from Gmail as usual. This n8n template adds it to your Google Sheet, follows up at most 3 times, and stops when the customer replies to your quote email. You never type a row.

- Picks up quotes by your Quote label or a PDF attachment, and saves each one to your sheet with its Gmail thread.
- Reads the customer, quote number and total from the PDF with Google Gemini, on quotes of 1 to 3 pages. A total is kept only if it appears word for word in the PDF. If not, the row goes to `review`, you get an email, and nothing is sent until you check it.
- Scanned PDFs, and anything it isn't sure is a quote, go to review and get no follow-up.
- Follows up on the days you choose (3, 4 and 7 days by default), only inside your sending hours. Never more than 3 follow-ups. Each one ends with "reply STOP".
- Any message from the customer in the quote's Gmail thread stops the follow-ups.

It does not send cold email, handle invoices or payments, or work outside Gmail. We don't offer installation, customization or calls.

You need your own n8n instance, Gmail and Google Sheets connected through your own Google Cloud app, and your own Gemini API key. On Google's free tier, Google may use and have people review what you send, so use a paid-tier Gemini key for real client quotes.

You get the workflow file (.json), a CSV with the column names for your Google Sheet, a setup guide, a changelog and the license terms (LICENSE.md). Delivered by Payhip as soon as you pay.

Support by email at smallhours.tools@gmail.com, in English. Replies come from an AI agent using fixed answers. There is no set reply time, so for a refund you don't need to wait for one. Bug fixes for 90 days after purchase. Full refund within 14 days of purchase, no questions asked. Email smallhours.tools@gmail.com from the address you bought with, and a person issues it; you don't need to wait for a written reply.

[Get it on Payhip: $19](https://payhip.com/b/OyN5V)

## Read more

Two short articles on the Small Hours blog, for people who build these flows for clients:

- [Detecting a client's reply in n8n: key on the Gmail thread ID, not a sheet column](https://smallhours.hashnode.dev/detecting-a-client-s-reply-in-n8n-key-on-the-gmail-thread-id-not-a-sheet-column): how to read the whole thread and tell the customer from yourself.
- [Replying in the same Gmail thread from n8n: threadId, Message-ID and the Re: subject](https://smallhours.hashnode.dev/replying-in-the-same-gmail-thread-from-n8n-threadid-message-id-and-the-re-subject): how to send the follow-up as a reply under the quote instead of a new email.

## License

The five free workflows in this repository are released under the MIT License (see `LICENSE`). The paid template has its own license, included with the purchase.
