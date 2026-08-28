---
name: gohighlevel
description: Working a limo operator's GoHighLevel CRM through Miles AI — new leads and response times, conversations awaiting a reply, follow-up candidates, pipeline and opportunity review, appointments, and the approval-gated send/tag/note/deal-update flow. Use whenever the question is about leads, contacts, conversations, deals, pipelines, appointments, or messaging a customer.
---

# GoHighLevel operations

Miles AI's GoHighLevel tools run on Limo Marketer's hosted server. They are
building blocks rather than fixed reports — most real questions are answered by
combining two or three of them, then reading an individual conversation.

The person asking is a limo operator or their dispatcher. They want to know who
to call, what to send, and whether the week is on track. They do not want an
analytics dump.

## Picking the right tool

| The question | Start with |
|---|---|
| "How did we do this week?" | `get_weekly_numbers` |
| "What leads came in, and did we answer them?" | `get_new_leads` |
| "How's our sales process doing?" | `run_sales_assessment` |
| "Who's waiting on us?" | `get_awaiting_reply` (mode `customer_waiting`) |
| "Who went quiet on us?" | `get_awaiting_reply` (mode `gone_quiet`) |
| "Who should we follow up with today?" | `get_followup_candidates` |
| "What are our biggest open deals?" | `list_opportunities` |
| "What does the pipeline look like?" | `get_pipeline_overview` |
| "What's on the calendar?" | `list_appointments` |
| "Look up this person" | `find_contact` |
| "What did we say to them?" | `get_conversation` |
| Anything the tools above don't cover | enumerate, then `get_conversation` per contact |

`get_conversation` is the universal fallback. When no single tool answers the
question, list the relevant contacts or deals and read the threads that matter,
one at a time.

## Scanning honestly

The list and scan tools work from a bounded slice, and they say so in their
output. Two rules:

- **When completeness matters, keep going.** Scans that stop early tell you
  exactly how to continue — an `offset` to resume from, or a narrower date
  range. Analytical tools take `start_date` + `end_date` for a deep scan of
  everything in that window instead of the quick default sample.
- **When you stop short, say what you skipped.** Never present a partial scan
  as a complete count. "Of the 50 most recent leads" is a fine answer;
  reporting 50 as the total when more exist is not.

Prefer narrowing the question over giving up on it.

## The follow-up run

`get_followup_candidates` is the whole workflow in one call. It returns open
deals with a future pickup date (or no pickup date on file — usually phone
bookings, included and flagged) that nobody has contacted recently, ranked by
value, each with a full dossier: contact details, pickup, cautions,
conversation excerpt, and trip details.

It deliberately returns a few more candidates than asked for, because some
should be skipped. Read each dossier before drafting:

- **Skip and say why** when the thread shows a complaint, a cancellation, a
  dispute, or anything that makes outreach unwelcome. Move to the next
  candidate — the spares exist for this.
- **Reference real context** when the conversation gives you something to work
  with. Fall back to the trip details when it doesn't.
- **Never invent** a price, a vehicle, or a commitment that isn't in the record.

## Before proposing any outreach

Check these every time. They are the difference between helpful and harmful:

1. **Do-not-disturb.** Contacts with DND set have opted out. Do not draft to
   them. `find_contact` and the follow-up dossiers both surface this.
2. **"Stop" replies.** If a contact ever replied stop, unsubscribe, or similar,
   they are done. Read the thread rather than assuming.
3. **Existing automations.** Run `list_workflows` and consider whether an
   active workflow is already messaging these people. Manual outreach on top of
   an automated sequence reads as spam and is the most common own-goal here.
4. **Recency.** Someone contacted yesterday usually should not be contacted
   again today.

## Writes are always two steps

Every change to GoHighLevel — sending a message, changing tags, adding a note,
creating or moving a deal — goes through the same gate, and it is not optional:

1. Call the `prepare_*` tool. It changes **nothing**. It returns a preview and
   a one-time `MILES-` code that expires in ten minutes.
2. **Show the operator the preview and wait for their explicit approval.**
3. Only then call `confirm_action` with the code.

**Never call `prepare_*` and `confirm_action` in the same turn.** The operator
has to see what will happen and say yes in between. If they decline or change
their mind, call `cancel_action` with the same code so it can never be used.

Codes are single-use and bound to that operator. A failed send burns the code —
prepare again rather than retrying the confirm.

Two prepare tools refuse by default and ask for an explicit override: moving a
deal that is already marked won or lost (`allow_reopen`), and creating a second
open deal for a contact who already has one (`allow_duplicate`). Do not set
either flag unless the operator has actually asked for that specific thing.

## Presenting results

- **Lead with the answer.** "Four people are waiting on you, longest since
  Tuesday" beats a table the operator has to read to find that out.
- **Name people and money.** Deal names, customer names, dollar values, and
  dates are what make a report actionable.
- **Say what to do next**, and keep it to the two or three things that matter
  most today.
- **Grades and scores are directional.** `run_sales_assessment` returns a
  scorecard; treat it as a prompt for a conversation, not a verdict.
- **Flag anything that looks wrong** in the data rather than smoothing over it —
  a missing pickup date, a deal with no value, a lead with no source.

## When something isn't working

`check_connection` is the diagnostic. It verifies the credential and that each
kind of data — contacts, conversations, opportunities, calendar, users — can
actually be read, in plain English. Run it whenever a tool reports a problem or
the setup is in question. Calendar and team reads are optional: if only those
fail, everything else still works and the report says so.
