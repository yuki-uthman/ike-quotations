# Ike Quotations

The follow-up list. Every quotation and sales order at MRH Investment that
still has money owing on it — what hasn't closed yet, so it can be chased.
Two cards: **Quotations** that were never confirmed, and **Unpaid orders**
that were. Click any row to see what's on it. Above them, a bar per day
showing how much still-open value was raised that day.

Like its siblings this repo is **static only**: no secret, no cron, no build
step. It fetches its data live from
[ike-data](https://github.com/yuki-uthman/ike-data), the shared Odoo pipeline
that also backs [ike-sales](https://github.com/yuki-uthman/ike-sales),
[ike-pos](https://github.com/yuki-uthman/ike-pos) (formerly ike-today) and
ike-expenses.

## What counts as still open

A record is on this page while **any part of its value is unsettled**, and it
leaves the moment the last rufiyaa arrives — whichever way it arrived.

| Stays | Leaves |
| --- | --- |
| Quotations (`draft` / `sent`) | Fully settled, by invoice **or** at the POS |
| Confirmed orders with a balance owing | Cancelled in Odoo (`cancel`) |
| Part-paid orders, showing the balance | Dismissed here as a dead lead |

**"Paid" is computed, not read, and that matters.** The obvious field lies in
both directions on this instance: `sale.order.invoice_ids` is empty on most
settled orders, and `invoice_status` reads "Fully Invoiced" on orders with no
linked invoice at all (S00203, for one). Money reaches an order two ways here
and each needs its own join — `account.move.invoice_origin`, which carries the
order name on 154 of 155 customer invoices, and
`pos.order.line.sale_order_line_id`, which is how the counter settles a
quotation against its order lines without ever creating a sale-order invoice.
So the number behind every row is

```
outstanding = amount_total − (paid invoice value + POS-settled value)
```

and a record is open while that is above a cent. Going by `invoice_status`
alone would have left 36 already-paid orders sitting here permanently.

Part payments fall out of this for free: the row shows the **remaining
balance** with the order total beneath it, tagged *Part paid*. The full
reasoning, and what's deliberately excluded (reversed invoices, draft POS
orders, section and note lines), lives in `ike-data/scripts/fetch_quotations.py`.

## Reading the page

- **The bar chart** is outstanding value by the date the quotation was raised,
  so a tall old bar is money that has been sitting a long time. Bars shrink as
  customers pay.
- **The page opens on today**, so the first thing you see is what came in
  today rather than three weeks of backlog. The day pills run oldest left,
  today right, and scroll underneath the pinned **`All`** pill on the left —
  one tap for the whole pipeline, every day at once.
- **Clicking a bar or a pill** narrows everything — total, both cards — to that
  one day. Clicking the selected bar again returns to `All`.
- **Each row** carries its reference, customer, the date and how long ago it
  was raised. A quotation within a week of its expiry date, or already past it,
  is flagged. Expanding a row shows the products, quantities and line totals,
  and who raised it.

## Dismissing a dead lead

Open a row and use **Not interested ×**. The record disappears from its card,
and its value comes out of the card total, the day total and that day's bar.
A counter appears at the bottom of the page; **Show** lists what's hidden and
restores any of it.

Two things this deliberately does *not* do:

- **It does not touch Odoo.** The quotation stays exactly as it is. This is a
  view-level "stop showing me this", not a cancellation.
- **It does not follow you.** The list lives in that one browser's
  `localStorage`. Dismiss something on your laptop and it is still there on
  your phone, and clearing site data brings everything back. That was the
  accepted trade for keeping this repo static — writing a dismissal back to
  Odoo needs a server, which none of these dashboards have.

## One-time setup

**Enable GitHub Pages**: Settings → Pages → Source: "Deploy from a branch" →
Branch: `main`, folder `/ (root)`. The page will be at
`https://<your-username>.github.io/ike-quotations/`.

That's all — the Odoo credential, the schedule and the settlement logic all
live in [ike-data](https://github.com/yuki-uthman/ike-data).

## How it gets its data

`index.html` fetches
`https://raw.githubusercontent.com/yuki-uthman/ike-data/main/data/quotations.json`
in the browser on every page load (`cache: 'no-store'`), then silently again
every 60 seconds and whenever you switch back to the tab — the underlying file
itself refreshes every 15 minutes. GitHub serves raw file content with
`Access-Control-Allow-Origin: *`, so this works cross-repo with no server or
API needed.

Open rows are remembered by reference across a refresh, so a new quotation
arriving does not slam shut the row you were reading. Unlike `sales.json`,
`quotations.json` keeps no history — it is a live snapshot of what is open
right now, and its day span shrinks as old quotations settle.

## Visibility

GitHub Pages on a free plan requires a **public** repository, so this page is
technically reachable by anyone with the exact URL, though it isn't linked or
indexed anywhere. Note this dashboard shows **customer names, order references
and amounts owing**, the same exposure as ike-pos. If that matters, GitHub
Pages on private repos requires GitHub Pro or higher.
