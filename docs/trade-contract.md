# Trade data contract

Before connecting an exchange feed, the producer and its consumers need to agree on what a trade record means. A record can have the right field types and still be unusable: an empty trade ID is a string, but it cannot reliably distinguish one execution from another.

This document defines the intended meaning of a normalized trade. It is a design reference, not a wire schema or evidence of a working live feed. The initial adapter will need to check the selected exchange's identifier scope, timestamp precision, and quantity conventions.

## Fields and meaning

| Field | Meaning | Intended rule |
| --- | --- | --- |
| Venue | Exchange that supplied the trade. | Present and nonempty. |
| Symbol | Trading pair, such as BTCUSDT. | Present and mapped to a known base and quote asset. |
| Trade ID | Exchange identifier for the execution. | Present and nonempty; interpreted within the exchange's documented scope. |
| Price | Quote asset paid per unit of base asset. | Positive exact decimal. |
| Quantity | Amount of base asset exchanged. | Positive exact decimal. |
| Trade time | Exchange-reported execution time. | Present, with an agreed time reference and precision. |

For BTC/USDT, BTC is the base asset and USDT is the quote asset. A trade of 0.002 BTC at 65,000 USDT per BTC has a quote value of 130 USDT before fees. Its base-asset volume is 0.002 BTC; those quantities measure different things.

The producer's receipt time is separate from trade time. Keeping both will help investigate delivery delays. Their difference alone is not a reliable network-latency measurement when clocks are not synchronized.

## Identity and repeated delivery

Price, quantity, and timestamp do not uniquely identify a trade. Two executions can share all three values, especially when a source records time at limited precision. Different trade IDs do not require different recorded timestamps.

For this invented example, assume trade IDs are unique within a venue and symbol:

| Delivery | Venue | Symbol | Trade ID | Quantity |
| --- | --- | --- | --- | --- |
| A | ExampleExchange | BTCUSDT | 1042 | 0.002 BTC |
| B | ExampleExchange | BTCUSDT | 1042 | 0.002 BTC |
| C | ExampleExchange | BTCUSDT | 1043 | 0.003 BTC |

A and B describe the same execution delivered twice. C describes another execution. The result is three received messages, two distinct trades, and 0.005 BTC of volume after deduplication. Deduplication means recognizing repeated deliveries so one execution contributes once.

Using just the trade ID across all markets and venues could incorrectly merge unrelated trades. The actual identity key must follow the selected exchange's documented scope. If repeated identities arrive with conflicting payloads, that requires investigation rather than silently treating them as harmless duplicates.

## Structure and validation

A schema describes fields and their types. The data contract also describes meanings, units, and validity rules. Decoding a message successfully does not establish that its price is positive or its identifier is usable.

The ingestion path will need to validate these rules before downstream calculations use a trade. Defining an identity rule does not itself implement deduplication: detecting repeats across a stream requires retaining identity state and deciding how long to keep it.

These rules are the starting point for reviewing the draft schema and validators. Live-adapter behavior, deduplication retention, and broker delivery guarantees still need implementation and verification in their respective milestones.
