# Before You Send: the live panel

Design as agreed on 1 October 2026. Decisions are recorded on the ground truth
chain in this repository (`.ground/truth/`); the IDs in brackets point to them.
Run `ground truth query active` to read the chain.

## Purpose

PR companies run their press releases and posts through Before You Send before
sending them [AX-OVK-00003]. The product shows a piece of content to a panel of
simulated people and reports how they respond, as individuals and as a group
[AX-OVK-00004].

Today the panel is static: personas are created for a test, read the content
cold, and are answered by one model. This design replaces the panels and
personas with a live engine. For now the existing report is kept and added to,
not redesigned [AX-OVK-00084, which supersedes AX-OVK-00013].

## The lens

We look at this through the eyes and lens of the person who is releasing the
content in order to communicate [AX-OVK-00076]. Panels, channels and data are
there to serve that person's view.

That person is two people [AX-OVK-00078], and they can want different things:

| Sender | Who | Cares most about | ID |
|---|---|---|---|
| The PR professional | Writes and sends the content, and uses the tool | Avoiding backlash, knowing how content lands, evidence to show a client | AX-OVK-00001 |
| The agency's client | The organisation whose message it is, such as a city government or a party | The public understanding or acting as intended, reputation | AX-OVK-00077 |

What the sender is trying to achieve is therefore an input alongside the
content: what they want people to understand, feel or do, and who it is aimed
at. The report judges whether the content arrived as intended
[AX-OVK-00079]. This rules out taking the content as the only input, and
reporting reactions without reference to what the sender intended.

We also look at this from the lens of the person being communicated to
[AX-OVK-00081]. Communication is decided at the receiving end: the sender
controls the words and the receiver controls the outcome. The receiver meets
the message in the middle of their own day, which is what the persona's pool,
the news and the other world inputs are there to represent.

The receiver is also two people [AX-OVK-00083]:

| | Sender | Receiver |
|---|---|---|
| First | The PR professional | The journalist |
| Final | The agency's client | The member of the public |

With both lenses, what the report measures is the gap between what the sender
intends and what the receiver takes away [AX-OVK-00082]. Each hop has its own
gap.

## Two kinds of content

Clients put two kinds of content through Before You Send [AX-OVK-00065]:

| | Social post | Press release |
|---|---|---|
| First reader | The public | A journalist or editor |
| What the public reads | The client's words | The journalist's coverage |
| Who decides if it is seen | The platform | The editor |
| Measure of success | Reactions, comments, shares | Whether it was covered, by whom, and how it was framed |

A press release reaches the public second-hand [AX-OVK-00066]. PR works in two
hops, not as a direct-to-consumer path [AX-OVK-00071]:

```
agency ──▶ journalist ──▶ coverage ──▶ the public
           will they     what they     reads the article,
           run it?       wrote         not the release
```

| Decision | Rules out | ID |
|---|---|---|
| Journalists are a panel, built as personas like any other. They are types, such as a business reporter at a national daily | Simulating named real journalists; testing releases on the public panel alone | AX-OVK-00072 |
| Coverage produced by the journalist panel goes onto flow and enters the public personas' streams as news | Treating the public's reaction to the release text as a prediction of public response | AX-OVK-00073 |

So a release is tested in two hops [AX-OVK-00075]. This uses machinery already
decided: a journalist is a persona on flow, and news articles are already an
input to the public personas [AX-OVK-00015].

| | A journalist persona |
|---|---|
| Identity | Outlet, beat, language, editorial stance |
| Stream | Incoming releases, plus the day's news |
| Pool | What has been covered this week and the running stories |
| Output | A decision to cover or not, and a draft article |

The rest of this document, the sample data and the onboarding standard cover
social posts only [AX-OVK-00067]. For press releases the report, the reality
data and the backtest are still open; see What is open.

## What is decided

| Decision | Rules out | ID |
|---|---|---|
| flow is the single event queue. The engine reads from it and writes to it; every other consumer reads through its own cursor | Components calling each other directly; results living only in job folders | AX-OVK-00009 |
| Each persona is a constantly running process that consumes flow, with its own stream of events and its own pool of knowledge | Personas created fresh per test; personas with no memory | AX-OVK-00010 |
| A persona that needs a judgement puts a job on flow. Multiple models, LLMs and SLMs, answer it, and the answers are aggregated into one response for that persona | One model for every persona; personas calling a model directly | AX-OVK-00011 |
| Prior posts and their real results form a reality dataset; predicted versus actual goes in an errors table | Reporting reactions with no check against real outcomes | AX-OVK-00012 |
| Personas are custom-built from statistical data about the real population | MatrAIx's ready-made personas; hand-written personas | AX-OVK-00014 |
| Inputs other than the content go into each persona's pool: news articles, time of day, weather, levels of attitudes such as sexism and racism | Feeding personas the content alone | AX-OVK-00015 |

Two further facts frame these. Before You Send is not static: a response depends
on the state of the world when the content is tested [AX-OVK-00005]. The finding
for a piece of content is the aggregate across the population, one response per
persona [AX-OVK-00006].

## How it works

The design follows the rapids, rivers and ponds pattern.

| Term | Here |
|---|---|
| Rapids | The flow queue: everything that happens |
| River (stream) | The slice of the queue one persona would notice |
| Pond (pool) | What one persona has taken in so far |

```
world inputs ──┐
(news, time,   │
 weather…)     ▼
content ──▶  flow ──▶ persona's stream ──▶ persona (always running)
               ▲                              │ takes inputs into its pool
               │                              │ on content: posts a job
               ├──────────── job ◀────────────┘
               │
               ├──▶ model A ─┐
               ├──▶ model B ─┼──▶ responses on flow
               ├──▶ SLM C  ──┘
               │
               ├──▶ aggregator: one response per persona
               ├──▶ aggregator: population result ──▶ Before You Send report
               └──▶ aggregator: result vs real outcome ──▶ errors table
```

The path of a piece of content is declared as seven behaviors
[AX-OVK-00023 to AX-OVK-00029].

### The boundary

```
agency ──▶  flow  ──▶ agency
 a job              a response
```

The engine's boundary is the flow queue. Content arrives from the agency on
flow and results are returned to flow. The engine returns its data and does
not decide what is shown [AX-OVK-00085]. This rules out the engine producing
the report itself, and holding back data because nothing uses it yet.

What is used from the returned data, and how it is presented, is worked out
after it is on the queue, by a customer advocate working from the customer's
side [AX-OVK-00086]. Three open questions therefore sit on that side and not
with the engine [AX-OVK-00087]: what the report shows for a press release,
whether clients are told which parts are checked, and how the report judges
against the sender's intent. How intent is captured as an input stays with the
engine.

A post is client specific. Content submitted for testing is a job for that
client, and the result is the response to that job [AX-OVK-00088]. Safeguards
that keep each client's jobs and responses separate are put in as the work
proceeds [AX-OVK-00089], under the rule that one client's unpublished draft
never reaches another client's panel [AX-OVK-00031].

### The response

The response to a job is the contract between the engine and whatever presents
it. It returns everything the engine produced, in structured form, with each
item marked as needed now or as extra [AX-OVK-00090]. It is made up of events
on flow that all carry the job ID: one per persona and one for the panel as a
whole [AX-OVK-00091].

The engine operates on the basis that, given what it is supplied, it does what
it can and sends back what is interesting. Whether any of it is used in a UI is
for others to choose [AX-OVK-00109]. A response states which inputs it was
given, what it could therefore produce, and what it could not and why. A
missing input reduces the response and never fails the job [AX-OVK-00110].

| Given | The engine can return |
|---|---|
| The content alone | Each persona's reaction, takeaway, next action, questions left, retelling, view of the sender; where the audience splits; the loud minority and the silent unhappy |
| + the sender's intent | The gap between what was meant and what arrived; the key point against where people stopped; who was missed |
| + the client's past posts and results | Most similar past posts and how they did; comparison with their history; how accurate the engine has been for them |
| + a news feed | What shaped each reaction; what the content collides with today |
| + the content is a press release | Whether journalists would cover it, how they would frame it, and the public's reaction to that coverage |
| + notice that it was published | The real result against the prediction, added to the errors table |

The current product is a crude prototype using one model. Its report is a
starting point for what is returned, not a requirement.

**Envelope, on every response** [AX-OVK-00092]: the job ID, the client, the
content exactly as tested, the content type, the sender's intent as given, the
panel and its composition, the models used, the log position the run used, the
time, the version of the response contract, and whether the run completed in
full, with what failed if it did not.

**Needed now.** What the current report draws.

| Level | Data | ID |
|---|---|---|
| Per persona | Age, gender, district, language; private reaction in their words; where they stopped reading; words that caught their attention; how they felt about each sentence; what they would say publicly and whether they would post it; anxiety score and negative-to-positive score; openness, agreeableness and worry | AX-OVK-00093 |
| Across the panel | Positive, neutral and negative counts; share who would post publicly; share worth a second look; reaction themes with counts | AX-OVK-00094 |

**Extra, from the two lenses.** Not asked for by the current report.

| Level | Data | ID |
|---|---|---|
| Per persona | The takeaway in one sentence; what they would do next; the questions they are left with; what was unclear or not in their language; whether it was relevant or useful to them; how they would retell it; what they now think of the sender | AX-OVK-00095 |
| Across the panel | The trait that best explains where the audience splits; the loud minority, who feel strongly and would post; the silent unhappy, who react badly and would say nothing; the distance between what each persona thought and what they would say | AX-OVK-00096 |

**About the result itself, on every response** [AX-OVK-00097]: each model's
answer and the merged answer for each persona; how far the models agreed; the
outcome of the character check; each persona's answer alone and in the group;
and for every item, its confidence and whether it can be checked against
reality.

**Returned when the inputs allow.** Declared as stages; read them as depending
on what is supplied [AX-OVK-00109].

| When | Data | ID |
|---|---|---|
| Once the reality dataset exists | The client's most similar past posts and how they did; what a simple rule would have predicted; how the draft compares with the client's history; later, matched by job ID, the real result and the error | AX-OVK-00098 |
| Once the sender's intent is an input | The gap between the intent and each takeaway; the sender's key point set against where people stopped; who was aimed at and did not notice or stopped early; who reacted without being aimed at | AX-OVK-00099 |
| As the larger parts are built | From the journalist panel: whether they would cover it and why, headline and angle, what they would cut and add, questions for the sender, the draft coverage. The gap at each hop and which of the sender's points survive. What in a persona's recent experience shaped its reaction. What the content collides with in the news and how it would land on another day. Whether it repeats or contradicts the client's earlier messages. Which sentence does the damage when removed. Who changed their mind in the group. Whether a rerun gives the same finding | AX-OVK-00100 |

**Rules and cautions.**

- A persona's private reaction and stop point are taken before any further
  question is put to it [AX-OVK-00101]
- The list of what is needed now, and the stages, are expected to change
  [AX-OVK-00102]
- Items in a response are plausible and not proven. A split found in a small
  panel can be noise, and a persona reporting on itself is the weakest kind of
  evidence [AX-OVK-00103]

### What makes up a persona

A working split, proposed and not on the chain:

| Layer | Examples | Changes |
|---|---|---|
| Identity | Age, district, language, personality | Never. Built from statistics |
| Attitudes | Trust in government, political leaning, prejudices | Slowly |
| Pool | News read, earlier posts from a client | Accumulates and fades |
| The moment | Time of day, weekday, weather | Current conditions only |

### Building personas from statistics

Sources named for Tallinn [AX-OVK-00008]: `arvudes.tallinn.ee` (official city
statistics, including resident satisfaction by district),
`strateegia.tallinn.ee` (the Tallinn 2035 strategy) and `digiriik.eesti.ee`
(a government e-service design toolbox; no population statistics were found on
its front pages).

Steps: collect the counts by district, age, gender and language; generate
individuals so the panel's proportions match the city's; reject impossible
combinations; add character; store each as a fixed identity.

Two gaps to plan for. Statistics say who people are, not how they think, so
attitudes need survey data or stated assumptions. And tables usually come one
dimension at a time, so combinations such as age by language by district need
a more detailed source or an assumption.

## What is open

Declared as unknowns, so nobody builds past them.

Design:

- How world inputs and attitude levels are represented, including whether a
  level of sexism or racism belongs to the persona, the public climate, or both
  [AX-OVK-00016]
- How the structured parts of several model responses are combined: by vote
  and middle value, or by a recorded random pick [AX-OVK-00060]

Data:

- How to check the group response against reality when client exports hold
  comment counts but no comment text [AX-OVK-00018]
- Whether other clients can supply the same export as the sample. Only one
  client's export has been seen [AX-OVK-00056]
- Whether a panel built from population statistics matches a client's actual
  followers. Follower demographics are not in the standard [AX-OVK-00057]
- The minimum number of posts a client needs for the backtest to mean anything
  [AX-OVK-00058]
- Whether Facebook should come first for every client. The priority rests on
  one client, a political party [AX-OVK-00061]

The response:

- The exact structure and field names [AX-OVK-00104]
- How confidence is calculated for each item [AX-OVK-00105]
- How many extra questions can be put to a persona before they change its
  answers or cost too much [AX-OVK-00107]
- How close the wording of a published post must be to a tested draft to count
  as the same post, and what happens when no match is found [AX-OVK-00113]

Press releases:

- What the report shows for a release. Likelihood of pickup, probable
  headlines by outlet and the hard questions have been named. These are
  additions to the existing report [AX-OVK-00084]. Owned by the customer
  advocate's side [AX-OVK-00074]
- What the reality data for press releases is, and whether clients can supply
  it. Media monitoring of coverage is the likely source and it is not in the
  sample [AX-OVK-00069]
- Which other channels matter. News site comment sections, email newsletters,
  LinkedIn and X have been named and not assessed [AX-OVK-00070]

The sender's intent:

- How intent is captured, how the report judges against it, and whose intent
  is used where the two senders differ. The input is an addition to the
  existing report [AX-OVK-00084]. Capturing intent is the engine's; judging
  against it in the report is the customer advocate's side [AX-OVK-00080]

Product:

- Whether clients are told which parts of the report are checked against
  reality and which are illustrative. Owned by the customer advocate's side
  [AX-OVK-00059]

The world model build also flagged that no declared behavior yet uses the
persona's pool when responding. That link depends on the first unknown.

## The bets

Declared as assumptions: believed, not proven.

| Bet | Risk | How to test | ID |
|---|---|---|---|
| A simulated panel predicts real audience reactions better than simple rules such as post type or caption length | High | Backtest on a client's past posts against baselines, scored on a held-out recent period | AX-OVK-00019 |
| World inputs in persona pools bring reactions closer to reality | Medium | Replay the news for the dates of past posts; compare error with and without | AX-OVK-00020 |
| Several models aggregated beat one model | Medium | Same backtest per model and aggregated | AX-OVK-00021 |
| Reaction, comment and share rates per person reached are an adequate measure of how a post landed | High | Where a client can supply comment text or survey results, compare them with the rates for the same posts | AX-OVK-00062 |
| Hides are a usable signal of negative reaction | Medium | Test whether posts rated high for backlash risk have higher hide rates, on the held-back quarter | AX-OVK-00063 |
| A caption can be tested on its own for photo posts, without the image | Medium | Compare backtest error on photo posts with error on text and link posts | AX-OVK-00064 |
| What a persona reports about itself, such as its takeaway, next action and retelling, is informative about real receivers | Medium | Compare the share whose next action is to comment or share with real comment and share rates for the same posts | AX-OVK-00108 |

The first is the leap of faith [AX-OVK-00022]. If a panel does not predict a
real audience, nothing else in the design matters.

## Onboarding data

Not all data holds the same weight, not all clients have the same data, and not
all platforms hold the same weight [AX-OVK-00037]. The same data is required
from every client [AX-OVK-00038], and the request is made by someone upstream
of this team, so it must be simple enough to pass along [AX-OVK-00039].

| Decision | Rules out | ID |
|---|---|---|
| The onboarding standard is the data present in the sample export. We work only with data we can actually get and accept the capabilities this removes | Asking for data beyond the standard export; designing around data clients cannot supply | AX-OVK-00040 |
| Text before video. Photo posts are included and tested on their caption | Video analysis in the first phase; excluding photo posts | AX-OVK-00041 |
| Facebook first, Instagram second | Starting with TikTok or YouTube; treating all platforms as equal | AX-OVK-00042 |
| The request is one instruction: a post-level export from Meta Business Suite for the Facebook page and Instagram account, covering the last 12 months | A custom request per client | AX-OVK-00043 |

### After a post goes out

The same export is requested again each quarter. Published posts are matched to
their jobs by their text. Where the published text differs from the tested
draft, the published text is run through the engine and that is what is scored
[AX-OVK-00111]. This is also how the engine learns that a draft was published,
and in which version [AX-OVK-00112]; it covers social posts and not press
releases.

The pull is not needed for the engine to run. It adds the one honest test:
a prediction made before the outcome existed. It also builds a track record
per client, keeps the calibration current, and lets published content enter
persona memory [AX-OVK-00030].

Three things to keep in mind when reading the results. People edit after seeing
a result, which is why the published text is rescored. Tested posts are not a
fair sample, since people test what worries them. And it is slow: one data
point per tested post, arriving quarterly.

### What is collected

| Tier | Data | ID |
|---|---|---|
| Must | Post text, publish date and time, post type, reach, reactions or likes, comment count, share count, post ID or link | AX-OVK-00044 |
| Should | Organic versus paid reach, hides, clicks. Facebook only in the sample. Never a condition of onboarding | AX-OVK-00045 |
| Could | Views; Instagram saves and follows | AX-OVK-00046 |
| Won't, for now | Comment text, reaction types, image files, follower demographics (not in the sample). Video metrics, viewer age and gender, country breakdown, TikTok and YouTube data (video is later) | AX-OVK-00047 |

Outcomes are measured per person reached, not as totals, because reach is set
by the platform and by paid promotion [AX-OVK-00048].

Two practical details of the Meta export: column names are in the client's
language, and the post text sits in either the title or the description column,
so both are read and each language needs a column mapping [AX-OVK-00055].

### The sample

One client's export, about 780 posts with outcomes over a year.

| | Facebook | Instagram |
|---|---|---|
| Posts | 491 | 290 |
| With a caption | 422 | 287 |
| Text or photo posts | 314 | 116 |
| Posts with any comments | 78% | 24% |
| Hides, clicks, paid reach | Yes | No |

On Facebook, reactions per 100 people reached run from about 0.9 to 3.6
between weak and strong posts, so there is a spread to predict. Only 2 posts
had paid reach.

## What the report can be checked against

The report is built from persona responses, not from client data, so under
these constraints it is still produced in full. Client data is used to check it
[AX-OVK-00049].

| Report section | Checkable? | ID |
|---|---|---|
| Would post publicly | Yes, against comment and share rates | AX-OVK-00050 |
| Strength of reaction (the quadrant's vertical axis) | Yes, against engagement per person reached | AX-OVK-00050 |
| Worth a second look | Partly, against clicks. Facebook only | AX-OVK-00050 |
| Sentiment split and backlash risk | Weakly. Hides are the only negative signal, on Facebook only, and reactions are not split by type | AX-OVK-00051 |
| Reaction themes, private reactions, where a persona stopped reading, attention words, the acceptability heatmap, the wording of public comments, reach | No | AX-OVK-00052 |
| The group response | Can be built and shown, not scored. Comment text is not collected | AX-OVK-00053 |

A panel percentage is not a forecast of a real rate. The report may say 25%
would post publicly where about 0.2 people per 100 reached really comment. The
claim the data supports is a comparison of a draft with the client's own past
posts [AX-OVK-00054].

## Operating rules

Recommended during design and accepted on 1 October 2026.

| Rule | Rules out | ID |
|---|---|---|
| A test must not change the audience. A draft is shown to a temporary copy of each persona, discarded afterwards. Only content a client really publishes enters memory | Personas remembering drafts they were tested on | AX-OVK-00030 |
| One client's unpublished draft must never reach a panel another client tests against | | AX-OVK-00031 |
| Each report records the point in the flow log it ran against, so it can be rerun exactly | Reports that cannot be reproduced | AX-OVK-00032 |
| A persona is idle until an event arrives. Small models decide whether it notices; larger models are used for reactions to drafts | Personas computing on a clock; a large model call for every event | AX-OVK-00033 |
| Every answer is checked for character breaks before it reaches a report: wrong language, details that contradict the identity, facts that contradict the content | Showing unchecked model answers to a client | AX-OVK-00034 |
| The most recent quarter of a client's data is held back for testing and never used for tuning. Posts published after the models' training cutoffs are preferred | Tuning and testing on the same posts | AX-OVK-00035 |
| Model responses are aggregated as structured reactions, and the persona's comment is one real model comment quoted word for word | Another model rewriting several answers into one | AX-OVK-00036 |

The last rule resolves the earlier unknown about aggregation [AX-OVK-00017].
How the structured parts are combined, by vote and middle value or by a
recorded random pick, is still to be chosen.

## Suggested order

1. Backtest the leap of faith: Facebook text and photo posts with captions,
   individual responses only, against simple baselines. This needs none of the live
   machinery and decides whether the rest is worth building.
2. Character check on persona answers.
3. Personas from statistics for one city.
4. Multiple models with aggregation.
5. The live loop: world inputs, pools, then the group response.

## What MatrAIx contributes

This repository is a fork of MatrAIx. Its ready-made personas are not used: its
schema has no Estonia, no Estonian language and nothing below "Eastern Europe".
Its useful parts are the persona generator with rules against impossible
combinations, the detailed persona schema, and the code that checks whether an
answer stayed in character.
