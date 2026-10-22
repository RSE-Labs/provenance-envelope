# Provenance envelope

A format for the facts about one payment, where every fact carries where it came from, how sure it is and the date the software learnt it. With it, an attrition record: what happens to those facts at each hop between the party that knows them and the figure a tax authority finally receives.

Version 0. Published by RSE Labs Ltd ([rse-labs.com](https://rse-labs.com)) as a proposal, not a standard. It was written for a reference ledger that runs on simulated data; it is not endorsed by HMRC, Open Banking or anyone else. Apache 2.0, so any originator, bank, platform, lender or software provider can adopt it, extend it or ignore it.

## Why

A bank line says an amount, a date and a reference. It does not say whether the car is on the mileage rate, what a platform payout was before the platform took its fee or how much of a mortgage payment was interest. Those facts live with the customer, the platform or the lender. Accounting software guesses at them or asks the customer. The envelope gives each such fact a place to travel beside the payment, with its provenance, so software can post a figure only when the evidence behind it is good enough, ask one question when a question would settle it and wait when nothing would.

## The envelope

`envelope.schema.json` is a JSON Schema (draft 2020-12). One envelope per payment:

- `transactionId`: the bank's identifier for the line.
- `envelopeVersion`: the version of this format the envelope follows.
- `bindings`: the records tied to the payment and how they were tied, for example a platform payout record bound by its payout identifier or a lender's instalment record bound by the mandate reference.
- `fields`: the facts, eighteen in version 0: the amount, booking date, direction, reference, descriptor, merchant name and category code; the gross figure, the fee and the VAT on it for a platform payout; the interest and the capital for a loan instalment; the business share, the owner's share, the vehicle and its method; the property; the kind of movement.

Every field carries a provenance class, a source and the date it arrived. A field left out does not apply to the payment; a field marked unknown applies but is not yet established. That distinction is what lets a ledger tell "not relevant" from "not known"; it is what the attrition record counts.

The five provenance classes, from strongest to weakest:

| Class | Meaning |
|---|---|
| stated | Sent by a party to the payment, such as the bank, the platform or the lender |
| derived | Computed from stated fields by a rule a tax authority or a regulator publishes |
| declared | Given by the customer or their agent in answer to a question, with the date they gave it |
| inferred | A best guess from indirect evidence such as a merchant category code |
| unknown | Not established yet |

The schema also defines the two originator records the envelope can bind in version 0: a platform payout record (consideration, fees, taxes, period and activity count, after the fields platforms already report to HMRC under the seller reporting rules) and a lender instalment record (instalment, interest, capital, rate and balance).

## The attrition record

`attrition.schema.json` describes what one payment's facts go through on the way to a tax return. Five hops:

1. `originator-record`: the platform's payout record or the lender's instalment record, where one exists.
2. `bank-rail`: what the bank carries, which is an amount, a date, a reference and some text.
3. `ledger-envelope`: the envelope the software assembled.
4. `ledger-posting`: the postings the software made from it.
5. `hmrc-boundary`: the figures in the cumulative update, where a self-employment update holds at most 33 numbers and a property update 16.

At each hop the record counts the fields and the pounds present by provenance class and lists what the hop added, folded into a coarser figure, moved to another period, dropped or held up. The boundary summary says how many envelope fields went in and which body fields the payment's pounds landed in. Read across a year of payments, it shows in pounds what each hop loses and what each new source would save.

## Versioning

Version 0 is the shape the reference ledger uses today. Changes that add fields or classes will bump the minor version, changes that alter the meaning of an existing field will bump the major version; both are listed in `CHANGELOG.md`. Proposals and corrections are welcome as issues or pull requests.

## Licence

Apache License 2.0. Copyright 2026 RSE Labs Ltd.
