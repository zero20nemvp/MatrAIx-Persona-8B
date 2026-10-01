# Before You Send, live panel: one-page summary

State of the design on 1 October 2026. Full version:
`before-you-send-live-panel.md`. Source of record: `ground truth query active`
(114 blocks).

## What it is

PR agencies send content through Before You Send before releasing it. A panel
of simulated people responds, and the result goes back to the agency. This
design replaces the panel behind the product with a live engine. The existing
report is kept and added to.

## The viewpoint

Two lenses, and the report measures the gap between them: what the sender
intends, and what the receiver takes away.

| | Sender | Receiver |
|---|---|---|
| First | The PR professional | The journalist |
| Final | The agency's client | The member of the public |

Clients test two kinds of content. A social post goes straight to the public.
A press release goes agency → journalist → coverage → public. Only social
posts are covered by data so far.

## How it works

- **flow is the one queue.** A job comes in from the agency; a response goes
  back. The engine does not decide what is shown; a customer advocate owns
  that side.
- **Personas run constantly.** Each has a stream of events and a pool of what
  it has taken in: news, time of day, weather, attitude levels.
- **Personas are built from statistics** about the real population.
  Journalists are a panel too, as types and never named people.
- **Several models answer each job**, merged into one response per persona.
- **A reality dataset** of past posts and real results gives an errors table.

## Rules

A test never changes the audience. One client's draft never reaches another
client's panel. Every report records the log position it ran against. Every
answer is checked for character breaks. The latest quarter of client data is
held back for testing.

## Client data

One request, passed along by someone upstream: a Meta Business Suite post-level
export, 12 months, Facebook first and Instagram second. Text before video.
Must have: post text, date, type, reach, reactions, comments, shares, post ID.
We work only with what clients can supply and accept what that removes.
The same export is requested again each quarter, to match published posts to
the drafts that were tested and score the predictions.

## The response

Given what it is supplied, the engine does what it can and sends back what is
interesting; a missing input reduces the response and never fails the job.
Everything the engine produced, structured, each item marked needed-now or
extra, with its confidence and whether it can be checked. Extras include the
takeaway, next action, how they would retell it, where the audience splits,
the loud minority and the silent unhappy. More arrives in stages as the reality
dataset, the intent input and the journalist panel come in.

## What can be checked against reality

Yes: would post publicly, strength of reaction. Weakly: sentiment and backlash
risk. No: themes, private reactions, wording of comments, the group response.
A panel percentage is not a forecast; the supportable claim is a comparison
with the client's own past posts.

## The bets

The leap of faith: a simulated panel predicts a real audience better than a
simple rule. Also unproven: that engagement rates measure how a post landed,
that world inputs and several models improve accuracy, and that what a persona
reports about itself reflects real people.

## Open, most important first

1. What the reality data for press releases is, and whether clients have it
2. How world inputs and attitude levels are represented
3. How the sender's intent is captured, and whose wins when the two differ
4. Whether other clients can supply the same export; only one has been seen
5. The structure of the response, and how confidence is calculated

Sixteen in all: thirteen with the engine, three with the customer advocate.

## First step

Backtest the leap of faith on the Facebook data in hand: posts with captions,
individual responses only, against simple baselines. It needs none of the live
machinery and decides whether the rest is worth building.
