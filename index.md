% Qrlew documentation master file.

# Qrlew Documentation

## What is Qrlew?

Qrlew is an [open source library](https://github.com/Qrlew) that can parse SQL queries into
Relations — an intermediate representation — that keeps track of rich data types, value
ranges, and row ownership; so that they can easily be rewritten into differentially-private
equivalent and turned back into SQL queries for execution in a variety of standard data
stores.

With Qrlew, a *data practitioner* can express their data queries in standard SQL; the *data
owner* can run the rewritten query without any technical integration and with strong privacy
guarantees on the output; and the query rewriting can be operated by a privacy-expert who
must be trusted by the owner, but may belong to a separate organization.

![Qrlew](./_static/qrlew_process.svg)

Qrlew provides the following features:

* **Qrlew provides automatic output privacy guarantees** With Qrlew a data owner can let
an analyst (data practitioner ) with no expertise in privacy protection run arbitrary SQL
queries with strong privacy garantees on the output.

* **Qrlew leverages existing infrastructures** Qrlew rewrites a SQL query into a differentially
private SQL query that can be run on any data-store with a SQL interface: from lightweight
DB to big-data stores. This removes the need for a custom execution engine and enables
differentially private analytics with virtually no technical integration.

* **Qrlew leverages synthetic data** Synthetic data are an increasingly popular way of privatizing a dataset. Using jointly: differentially
private mechanisms and differentially private synthetic data can be a simple, yet powerful,
way of managing a privacy budget and reaching better utility-privacy tradeoffs.

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