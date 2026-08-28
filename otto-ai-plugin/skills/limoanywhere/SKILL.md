---
name: limoanywhere
description: Reading a limo operator's LimoAnywhere back office through Otto AI — the trip calendar and daily schedule, quote requests and their conversion, reservations across the new/online/unfinalized/deleted screens, and booked revenue. Use whenever the question is about trips, jobs, runs, quotes, reservations, confirmation numbers, drivers, vehicles, or booked revenue.
---

# LimoAnywhere operations

These tools read the operator's LimoAnywhere back office. They run **on this
machine**, signed in as the operator's own LimoAnywhere user — so what you can
see is exactly what they'd see in a browser.

**Everything here is read-only.** Nothing in this toolset can convert a quote,
save a reservation, assign a driver, or delete anything. You never need to ask
permission before reading, and you should never imply a change was made. If the
operator wants something changed, tell them plainly that Otto can't do it and
they'll need to do it in LimoAnywhere directly.

LimoAnywhere is the *operations* side (actual trips and money). If the
session also has Otto AI's GoHighLevel connector — the *marketing* side
(leads, conversations, pipeline) — many good answers combine both; if it
doesn't, stay on the operations side rather than guessing at marketing data.

## Picking the right tool

| The question | Start with |
|---|---|
| "What's on today / tomorrow / this weekend?" | `la_get_schedule` |
| "How many quotes came in?" | `la_list_quotes` |
| "Tell me about quote 97751" | `la_get_quote` |
| "What reservations do we have?" | `la_list_reservations` |
| "Pull up confirmation 97362" | `la_get_reservation` |
| "Which quotes did we lose?" | `la_quote_conversion_report` |
| "How much is booked this month / next month?" | `la_revenue_summary` |
| Setup or something's broken | `la_check_connection` |

`la_get_schedule` and `la_revenue_summary` both work for **future** dates. "How
much is on the books for next month?" is a normal, answerable question.

## Two quirks that change answers

**Quotes and reservations are filtered by different dates.** `la_list_quotes`
works by *when the quote was requested*. `la_list_reservations` and
`la_revenue_summary` work by *pickup date*. "Quotes last week" and "trips last
week" are different windows over different things — be explicit about which one
you answered, because operators ask for both using the same words.

**Reservations live on four separate screens.** `list_from` picks which:

- `new_reservations` — the main list, and the right default
- `online` — web bookings and eFarm-in that may be *awaiting acceptance*
- `unfinalized` — started but not completed
- `deleted` — removed reservations

A trip missing from the main list may simply be on another screen. "Any online
bookings we haven't accepted?" means `list_from: "online"`. When looking up a
specific confirmation number, `la_get_reservation` already searches all four.

## Scanning honestly

List tools walk a bounded number of pages and say so when they stop early. If a
scan truncates, **narrow the date range and say what you covered** — never
present a partial list as a complete count. The operator is often asking
because a number matters for a decision.

## Quote conversion is a heuristic

`la_quote_conversion_report` matches quotes to reservations by passenger name
plus pickup date. That is genuinely useful and genuinely fallible: a trip
rebooked under a different name, a changed pickup date, or a group booking can
all break the match.

Present the results as **leads to check, never as verdicts**. "These eight look
unconverted — worth a call" is right. "We lost eight quotes" is not. Spot-check
anything surprising with `la_get_quote` and `la_get_reservation` before the
operator acts on it.

## Talking about money

- Totals are reservation **grand totals**. Farm-in/out costs and settlements are
  not netted out, so this is booked revenue, not profit.
- **Cancelled and no-show trips are reported separately** and excluded from
  booked totals. If an operator's number disagrees with yours, this is usually
  why — say which basis you used.
- `la_get_reservation` gives the full money picture for one trip: grand total,
  payments and deposits taken, and the balance still due.

## Presenting results

- **Lead with the answer.** "Six runs tomorrow, first pickup 5:40am" beats a
  table they have to scan.
- **Schedules read chronologically**, grouped by day, with pickup time,
  passenger, and vehicle. That's how a dispatcher thinks.
- **Use confirmation numbers.** They're how the operator finds the trip in their
  own system, so include them whenever you name a trip.
- **Flag the operationally interesting things** without being asked: an
  unassigned driver on a trip tomorrow, an unaccepted online booking, a large
  balance due on a job about to run.

## When something isn't working

`la_check_connection` verifies the saved login works and that both quotes and
the calendar can be read. If the login is being rejected, LimoAnywhere is
usually the cause — a changed password, or a user whose permissions were
reduced. The fix is `/otto-setup` again with current credentials.

LimoAnywhere is an older system and does go down; `la_check_connection`
distinguishes "our login is wrong" from "LimoAnywhere is having trouble."
