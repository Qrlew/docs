% Qrlew documentation master file.

# Qrlew Documentation

## What is Qrlew?

Qrlew is an [open source library](https://github.com/Qrlew) that can parse SQL queries into
*Relations* — an intermediate representation — that keeps track of rich data types, value
ranges, and row ownership; so that they can easily be rewritten into [differentially-private](https://differentialprivacy.org/resources/)
equivalent and turned back into SQL queries for execution in a variety of standard data
stores.

With Qrlew, a [data practitioners](/definitions.md#data-practitioner) can express their data queries in standard SQL; the *data
owner* can run the rewritten query without any technical integration and with strong privacy
guarantees on the output; and the query rewriting can be operated by a privacy-expert who
must be trusted by the owner, but may belong to a separate organization.

:::{figure-md}
![Qrlew](./_static/qrlew_process.svg)

The rewriting process occurs in three stages: The [data practitioners](#data-practitioner)’s query is parsed
into a Relation, which is rewritten into a DP equivalent and finally executed by the the data
owner which returns the privacy-safe result.
:::


## Main Features

Qrlew provides the following features:

* **Qrlew provides automatic output privacy guarantees** With Qrlew a data owner can let
an analyst ([data practitioners](#data-practitioner)) with no expertise in privacy protection run arbitrary SQL
queries with strong privacy garantees on the output.

* **Qrlew leverages existing infrastructures** Qrlew rewrites a SQL query into a differentially
private SQL query that can be run on any data-store with a SQL interface: from lightweight
DB to big-data stores. This removes the need for a custom execution engine and enables
differentially private analytics with virtually no technical integration.

* **Qrlew leverages synthetic data** Synthetic data are an increasingly popular way of privatizing a dataset. Using jointly: differentially
private mechanisms and differentially private synthetic data can be a simple, yet powerful,
way of managing a privacy budget and reaching better utility-privacy tradeoffs.

## How does it work?

In this chapter the following [definitions](/definitions.md) will hold.

### Design Goals


Qrlew assumes the *central model of differential privacy* [^1], where a trusted central organization: hospital, insurance company, utility provider, called the *data owner*, collects and stores personal data in a secure database and whishes to let untrusted [data practitioners](#data-practitioner) run SQL queries on its data.

[^1]: Arvind Narayanan and Vitaly Shmatikov. Robust de-anonymization of large sparse datasets. In 2008 IEEE Symposium on Security and Privacy (sp 2008), pages 111–125. IEEE, 2008.

At a high level we pursued the following requirements:

* Ease of use for the [data practitioners](#data-practitioner). The [data practitioners](#data-practitioner) are assumed to be a data experts but no privacy experts. They should be able to express their queries in a standard way. We chose SQL as the query language as it is very commonly used for analytics tasks.
* Ease of integration for the *data owner*. As SQL is a common language to express data analysis tasks, many data-stores support it from small embedded databases to big data stores.
* Simplicity for the *data owner* to setup privacy protection. Differential privacy is about capping the sensitivity of a result to the addition or removal of an individual that we call *privacy unit*. \qrlew{} assumes that the *data owner* can tell if a table is public and, if it is not, that it can assign exactly one \emph{privacy unit} to each row of data. In the case there are multiple related tables, \qrlew{} enables to define easily the \emph{privacy units} for each tables transitively.
* Simple integration with other privacy enhancing technologies such as *synthetic data*. To avoid repeated privacy losses or give result when a DP rewriting is not easily available (e.g. when the query is: \texttt{SELECT * FROM table}) \qrlew{} can use *synthetic data* to blend in the computation.


These requirements dictated the overall \emph{query rewriting} architecture and many features, the most important of which, are detailed below.


### SQL Query IR
[Qrlew](https://qrlew.github.io/) transforms a SQL query into a combination of simple operations such as Map, Reduce and Join that are applied to Tables. This representation simplifies the process of rewriting queries and reduces dependencies on the diverse range of syntactic constructs present in SQL.

### Type Inference Engine
Differential Privacy (DP) guarantees are hard to obtain without destroying too much information. In many mechanisms having prior bounds on values can improve the utility of DP results dramatically. By propagating types cleverly, [Qrlew](https://qrlew.github.io/) can returns bounds for all values.

- {doc}`tutorials/range_propagation`
- {doc}`tutorials/dataset_from_queries`
- {doc}`tutorials/rewrite_with_dp`

### Differential Privacy Rewriter
[Qrlew](https://qrlew.github.io/) can compile SQL queries into Differentially Private ones. The process is inspired by Wilson et al. 2020. The complexity of the compilation process makes [Qrlew](https://qrlew.github.io/) IR very useful at delivering clean, readable and reliable code.

[White paper](https://hal.science/hal-04350665v1/document)

## Contributing

To contribute to this code, follow the [instructions](contributing)

# Qrlew white paper

Read [Qrlew white paper](https://hal.science/hal-04350665v1/file/qrlew.pdf).

And cite us:
```bibtex
@article{grislain2023qrlew,
  title={Qrlew: Rewriting SQL into Differentially Private SQL},
  author={Grislain, Nicolas and Roussel, Paul and de Sainte Agathe, Victoria},
  url={https://hal.science/hal-04350665},
  year={2023},
  month=Dec,
  keywords={SQL ; Privacy ; Differential Privacy ; Rewritting system ; Rust ; Privacy Enhancing Technology ; Privacy by Design},
  pdf={https://hal.science/hal-04350665/file/qrlew.pdf},
  hal_id={hal-04350665},
  hal_version={v1},
}
```

# Indices and tables

- {ref}`genindex`
- {ref}`modindex`
- {ref}`search`

```{toctree}
---
maxdepth: 2
hidden:
titlesonly:
---
tutorials/getting_started
tutorials/user_guide
api
```