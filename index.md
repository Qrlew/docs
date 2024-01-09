% Qrlew documentation master file.

# Qrlew Documentation


```{toctree}
---
maxdepth: 2
caption: Contents:
---
tutorials/getting_started
tutorials/user_guide
api
```
## What is [Qrlew](https://qrlew.github.io/)?

[Qrlew](https://qrlew.github.io/) is an open source library that can parse SQL queries into
Relations — an intermediate representation — that keeps track of rich data types, value
ranges, and row ownership; so that they can easily be rewritten into differentially-private
equivalent and turned back into SQL queries for execution in a variety of standard data
stores.
With Qrlew, a data practitioner can express their data queries in standard SQL; the data
owner can run the rewritten query without any technical integration and with strong privacy
guarantees on the output; and the query rewriting can be operated by a privacy-expert who
must be trusted by the owner, but may belong to a separate organization.

[Qrlew](https://qrlew.github.io/) is an open source library that aims to parse and translate SQL queries into an Intermediate Representation (IR) that is well-suited for various rewriting tasks. Although it was originally designed for privacy-focused applications, it can be utilized for a wide range of purposes.

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

# Indices and tables

- {ref}`genindex`
- {ref}`modindex`
- {ref}`search`
