---
name: trader-broker
description: >-
  Order Ticket for stocks. Turns an order that /trader-risk passed into a
  plain ticket the user types into their own broker (Alpaca, Interactive
  Brokers or any other), one order at a time, records the user's decision
  word for word, and records the fill as a tax lot afterwards. Claude never
  places the order. Use when the user says "place the orders", "buy it",
  "show me the tickets", "what do I type in my broker", "I bought it", or
  after /trader-risk passes.
argument-hint: "[order number]"
---

# trader-broker

The last step for stocks. It does not connect to any broker: it shows each
order that passed `/trader-risk` as a ticket, and **the user places it
themselves** in their broker's app or website. Claude never calls an order
tool, even when one is connected and even when the user says yes.

`${CLAUDE_SKILL_DIR}` below is this skill's folder (where this SKILL.md lives).

```bash
python3 "${CLAUDE_SKILL_DIR}/ticket.py" show                     # every ticket, READY or not
python3 "${CLAUDE_SKILL_DIR}/ticket.py" approve --index 1 --reply "yes"
python3 "${CLAUDE_SKILL_DIR}/ticket.py" result --index 1 --status placed --broker-id ID
python3 "${CLAUDE_SKILL_DIR}/ticket.py" fill --symbol AAPL --side buy --qty 3.6628 --price 341.27
python3 "${CLAUDE_SKILL_DIR}/ticket.py" log
```

`ticket.py` never talks to a broker. It renders tickets, checks that a reply
is a real yes for that order, and keeps the log in `trader/state/broker.json`.

## Steps, for each order

1. `ticket.py show`. Only READY tickets go to the user. A ticket that is NOT
   READY or BLOCKED goes back to `/trader-risk`.
2. Show ONE ticket in plain words (side, amount or shares, symbol, order type,
   time in force) and ask whether they want to place it themselves. One order
   per question. Never "yes to all".
3. Record the reply word for word: `ticket.py approve --index N --reply "<reply>"`.
   If it prints "Not recorded", the reply was not a clear yes for this order:
   ask again. Never turn a reply into a yes yourself.
4. The user places the order in their broker. Claude does not, and does not
   offer to.
5. When the user says what happened, record it: `ticket.py result --index N
   --status placed --broker-id <id if they have it>`, or `--status denied` if
   they skipped it, or `--status failed --note "..."`.
6. When the user gives the fill (shares and price), `ticket.py fill` adds it
   to `trader/portfolio.json` as a tax lot for `/trader-tax`. Then the next ticket.

## What it never does

- Never places, changes, cancels or closes an order, and never calls a broker's order tools.
- Never acts on an order from a scheduled or background task.
- Never asks for, reads, stores or repeats API keys or passwords.

## Safety

- Education and tooling, not financial advice. No order here is a recommendation, and nothing promises returns.
- Paper trading first (`"paper": true`). Going live is the user's own decision.
- An order is only shown when trader-risk passed that exact order today and the user said yes to that
  one order in the chat; the user then places it themselves.
- No options, margin, short selling or leverage unless the config allows it and the user confirms.
- Broker keys stay with the user and their broker, never in the chat.
