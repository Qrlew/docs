---
title: Qrlew Documentation
---

# Qrlew

## The open-source DP layer for SQL

[Qrlew](https://qrlew.github.io/) (/ˈkɝlu/) is the [open source library](https://github.com/Qrlew) that rewrites SQL queries into privacy-preserving variants using [Differential Privacy (DP)](https://en.wikipedia.org/wiki/Differential_privacy).

Use Qrlew if you want to bring privacy guarantees to your SQL pipelines. It is:
- **SQL-to-SQL**: Qrlew turns SQL queries into differentially-private SQL queries that can be executed at scale on many SQL datastore, in many SQL dialects.
- **Feature-rich**: Qrlew covers the broadest range of SQL queries, including JOIN and nested queries
- **Privacy-optimized**: Qrlew keeps track of tight bounds and ranges throughout each step, minimizing the amount of noise needed to achieve differential privacy.

```{figure} ./_static/qrlew_process.svg
:name: fig_qrlew_process

The rewriting process occurs in three stages: The [data practitioners](/definitions.md#data-practitioner)’s query is parsed
into a Relation, which is rewritten into a DP equivalent and finally executed by the the data
owner which returns the privacy-safe result.
```

## Qrlew motivations: make differential privacy affordable for analytics use cases

[Qrlew](https://qrlew.github.io/) assumes the *central model of differential privacy* {cite}`archie2018s`, where a trusted central organization: hospital, insurance company, utility provider, called the [data owner](/definitions.md#data-owner), collects and stores personal data in a secure database and whishes to let untrusted [data practitioners](/definitions.md#data-practitioner) run SQL queries on its data.

At a high level we pursued the following requirements:

* Ease of use for the [data practitioners](#data-practitioner). The [data practitioners](#data-practitioner) are assumed to be data experts but no privacy experts. They should be able to express their queries using the most common dialect for data analysis: SQL.
* Ease of integration for the [data owner](/definitions.md#data-owner). SQL is a common language to express data analysis tasks supported by most datastores of all scale.
* Simplicity for the [data owner](/definitions.md#data-owner) to setup privacy protection. Differential privacy is about capping the sensitivity of a result to the addition or removal of an individual that we call [privacy unit](/definitions.md#datasets-and-privacy-units-pu). [Qrlew](https://qrlew.github.io/) assumes that the [data owner](/definitions.md#data-owner) can tell if a table is public and, if it is not, that it can assign exactly one [privacy unit](/definitions.md#datasets-and-privacy-units-pu) to each row of data. In the case there are multiple related tables, [qrlew](https://qrlew.github.io/) enables to define easily the [privacy units](/definitions.md#datasets-and-privacy-units-pu) for each table transitively.
* Simple integration with [synthetic data](/definitions.md#synthetic-data-sd) when available. Some queries are not very suitable for DP-rewriting (e.g.: `SELECT * FROM table`), in those cases [qrlew](https://qrlew.github.io/) can use [synthetic data](/definitions.md#synthetic-data-sd) as a fallback if provided.

These requirements dictated the overall *query rewriting* architecture and features.

## How does [Qrlew](https://qrlew.github.io/) work?

The [Qrlew](https://qrlew.github.io/) library, solves the problem of running a SQL query with [DP](/definitions.md#differential-privacy-dp) guarantees in three steps:
1. the SQL query submitted by the [data practitioners](#data-practitioner) is parsed and converted into an intermediate representation called [Relation](https://en.wikipedia.org/wiki/Relation_(database)), this [Relation](https://en.wikipedia.org/wiki/Relation_(database)) is designed to ease the tracking of data types ranges or possible values, to ease the tracking of the [privacy unit](/definitions.md#datasets-and-privacy-units-pu) in the next step. 
2. The [Relation](https://en.wikipedia.org/wiki/Relation_(database)) is rewritten into a DP variant
3. The DP variant of the [Relation](https://en.wikipedia.org/wiki/Relation_(database)) can be rendered as an SQL query string in any dialect.

At the end of this process, the query string can submitted to the data store of the *data owner*. The output can be shared with the *data practitioner* or published without worrying about privacy leakage. This process is illustrated in {numref}`fig_qrlew_process`.

# Deep Dive into Qrlew

To learn more about qrlew read the [deep-dive](/deep_dive.md) section, or read [Qrlew white paper](https://arxiv.org/pdf/2401.06273.pdf).

You can cite us:
```bibtex
@misc{grislain2024qrlew,
  title={Qrlew: Rewriting SQL into Differentially Private SQL}, 
  author={Nicolas Grislain and Paul Roussel and Victoria de Sainte Agathe},
  year={2024},
  eprint={2401.06273},
  archivePrefix={arXiv},
  primaryClass={cs.DB},
  url={https://arxiv.org/pdf/2401.06273.pdf},
}
```

# Tutorials

- {doc}`tutorials/range_propagation`
- {doc}`tutorials/dataset_from_queries`
- {doc}`tutorials/rewrite_with_dp`

# Definitions

Useful *definitions* can be found [there](/definitions.md).

# Contributing

To contribute to this code, follow the [instructions](/contributing.md)

```{toctree}
---
maxdepth: 2
hidden:
titlesonly:
---
tutorials/getting_started
tutorials/user_guide
api
contributing
```
