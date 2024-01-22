---
title: Rewriting rules
orphan: true
---
# Rewriting rules
## Relation

A [Relation](/definitions.md#relation) is an intermediate representation of a query that allow to easily handle DP rewriting by making it easy:
- track the privacy unit,
- and estimate the sensitivity of the query.

Any SQL query can be transformed into a [Relation](/definitions.md#relation).

[Relations](/definitions.md#relation) can be have different *properties* and be rewritten into other [Relations](/definitions.md#relation) with specified properties.

This page lists the different properties and the different rewriting rules.

## Properties

Each *Relation* can have one of the following properties:
- `Public`: A relation derived from public tables is labeled as such and does not require any further protection to be disclosed.
- `Published`: A relation is considered Published if its input relations are either Public, DP, in some cases SD, or Published themselves. It can be considered as Published but with some more care like the need to account for the privacy loss incurred by its DP ancestors.
- `Private`: Cannot be disclosed.
- `PrivacyUnitPreserving` (PUP): A *Relation* is PUP if each row is associated with a PU. In practice it will have a column containing the PID identifying the PU
- `DifferentiallyPrivate`: A *Relation* will be DP if it implements a DP mechanism. A DP *Relation* can be safely  executed on private data and the result be published. Note that the *privacy loss* associated with the DP mechanism has to be accurately accounted for (see section~\ref{sec:privacy_analysis}).
- `SyntheticData`: In some contexts a *synthetic data* version of source tables is available. Any *Relation* derived from other SD or Public *Relations* is itself SD.


### Table
A `Relation::Table` may be:
- `Private`
- `PrivacyUnitPreserving`
- `Public`
- `SyntheticData`

## Rewriting rules

The Rewriting rules are [defined there](https://docs.rs/qrlew/latest/src/qrlew/rewriting/rewriting_rule.rs.html#602-850).

They can be summarized as follow.

### Map
| if its input is | the *Map* can be rewritten into |
|:----------|:----------|
| `Public` | `Public` |
| `Published` | `Published` |
| `DifferentiallyPrivate` | `Published` |
| `PrivacyUnitPreserving` | `PrivacyUnitPreserving` |
| `SyntheticData` | `SyntheticData` |

### Reduce
| if its input is | and if | the *Reduce* can be rewritten into |
|:----------|:----------| :----------|
| `Public` | | `Public` |
| `Published` | | `Published` |
| `SyntheticData` | | `SyntheticData` |
| `PrivacyUnitPreserving` | all the aggregations are DP-rewritable |`DifferentiallyPrivate` |

### Join
| if its left input is | if its right input is | the *Join* can be rewritten into |
|:----------|:----------|:----------|
| `Public` | `Public` | `Public` |
| `Published` | `Published` | `Published` |
| `Published` | `PrivacyUnitPreserving` | `PrivacyUnitPreserving` |
| `PrivacyUnitPreserving` | `Published` | `PrivacyUnitPreserving` |
| `PrivacyUnitPreserving` | `PrivacyUnitPreserving` | `PrivacyUnitPreserving` |
| `DifferentiallyPrivate` | `PrivacyUnitPreserving` | `PrivacyUnitPreserving` |
| `PrivacyUnitPreserving` | `DifferentiallyPrivate` | `PrivacyUnitPreserving` |
| `SyntheticData` | `SyntheticData` | `SyntheticData` |

### Set
| if its left input is | if its right input is | the *Set* can be rewritten into |
|:----------|:----------|:----------|
| `Public` | `Public` | `Public` |
| `Published` | `Published` | `Published` |
| `Published` | `PrivacyUnitPreserving` | `PrivacyUnitPreserving` |
| `SyntheticData` | `SyntheticData` | `SyntheticData` |

### Values
A `Relation::Values` may be:
- `Public`
- `SyntheticData`


## Rewriting
The rewriting process consists of three sequential steps:

1. **Initial Assignment**: Each [Relation](/definitions.md#relation) is assigned a set of rules based on its type. For instance, if it contains only supported aggregations, a `Reduce` may be categorized as `Public`, `Published`, `SyntheticData`, or `DifferentiallyPrivate`.

2. **Bottom-Up Rule Selection**: Starting from the bottom of the tree, rules are selected based on the possible rules of the input(s). For example, if the input of a `Reduce` is `Public`, then the `Reduce` can only be categorized as `Public`. The other properties are eliminated during this step.

3. **Scoring**: Each property is assigned a score:
    - `Public`: 10
    - `Published`: 1
    - `Private`: 0
    - `PrivacyUnitPreserving`: 2
    - `DifferentiallyPrivate`: 5
    - `SyntheticData`: 1
The higher the score, the more desirable the property. The score of a graph of relations with properties is the sum of the scores of all the relations. The combination of rules that maximizes the total score is selected.

This process is explained in more details in the [deep dive section](/deep_dive.md#privacy-properties-and-rewriting-rules).