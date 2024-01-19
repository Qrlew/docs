---
title: Deep Dive
---

# Deep Dive

## What is Qrlew?

[Qrlew](https://qrlew.github.io/) is an [open source library](https://github.com/Qrlew) that can parse SQL queries into
[Relations](#qrlew-intermediate-representation) — an intermediate representation — that keeps track of rich data types, value
ranges, and row ownership; so that they can easily be rewritten into [differentially-private](/definitions.md#differential-privacy-dp)
equivalent and turned back into SQL queries for execution in a variety of standard data
stores.

With [Qrlew](https://qrlew.github.io/), a [data practitioner](/definitions.md#data-practitioner) can express their data queries in standard SQL; the [data owner](/definitions.md#data-owner) can run the rewritten query without any technical integration and with strong privacy
guarantees on the output; and the query rewriting can be operated by a privacy-expert who
must be trusted by the owner, but may belong to a separate organization.

```{figure} ./_static/qrlew_process.svg
:name: fig_qrlew_process

The rewriting process occurs in three stages: The [data practitioner](/definitions.md#data-practitioner)’s query is parsed
into a Relation, which is rewritten into a DP equivalent and finally executed by the the data
owner which returns the privacy-safe result.
```

## Main Features

[Qrlew](https://qrlew.github.io/) provides the following features:

* **[Qrlew](https://qrlew.github.io/) provides automatic [output privacy](/definitions.md#privacy-enhancing-technologies-pet) guarantees:** With [Qrlew](https://qrlew.github.io/) a [data owner](/definitions.md#data-owner) can let
an analyst ([data practitioner](/definitions.md#data-practitioner)) with no expertise in privacy protection run arbitrary SQL
queries with strong privacy guarantees on the output.

* **[Qrlew](https://qrlew.github.io/) leverages existing infrastructures:** Qrlew rewrites a SQL query into a differentially
private SQL query that can be run on any data-store with a SQL interface: from lightweight
DB to big-data stores. This removes the need for a custom execution engine and enables
differentially private analytics with virtually no technical integration.

* **[Qrlew](https://qrlew.github.io/) leverages [synthetic data](/definitions.md#synthetic-data-sd):** [Synthetic data](/definitions.md#synthetic-data-sd) are an increasingly popular way of privatizing a dataset. Using jointly: differentially
private mechanisms and differentially private [synthetic data](/definitions.md#synthetic-data-sd) can be a simple, yet powerful,
way of managing a privacy budget and reaching better utility-privacy tradeoffs.

## Design Goals

[Qrlew](https://qrlew.github.io/) assumes the *central model of differential privacy* {cite}`archie2018s`, where a trusted central organization: hospital, insurance company, utility provider, called the [data owner](/definitions.md#data-owner), collects and stores personal data in a secure database and whishes to let untrusted [data practitioners](/definitions.md#data-practitioner) run SQL queries on its data.

At a high level we pursued the following requirements:

* Ease of use for the [data practitioners](#data-practitioner). The [data practitioners](#data-practitioner) are assumed to be a data experts but no privacy experts. They should be able to express their queries in a standard way. We chose SQL as the query language as it is very commonly used for analytics tasks.
* Ease of integration for the [data owner](/definitions.md#data-owner). As SQL is a common language to express data analysis tasks, many data-stores support it from small embedded databases to big data stores.
* Simplicity for the [data owner](/definitions.md#data-owner) to setup privacy protection. Differential privacy is about capping the sensitivity of a result to the addition or removal of an individual that we call [privacy unit](/definitions.md#datasets-and-privacy-units-pu). [Qrlew](https://qrlew.github.io/) assumes that the [data owner](/definitions.md#data-owner) can tell if a table is public and, if it is not, that it can assign exactly one [privacy unit](/definitions.md#datasets-and-privacy-units-pu) to each row of data. In the case there are multiple related tables, [Qrlew](https://qrlew.github.io/) enables to define easily the [privacy units](/definitions.md#datasets-and-privacy-units-pu) for each tables transitively.
* Simple integration with other privacy enhancing technologies such as [synthetic data](/definitions.md#synthetic-data-sd). To avoid repeated privacy losses or give result when a DP rewriting is not easily available (e.g. when the query is: `SELECT * FROM table`) [Qrlew](https://qrlew.github.io/) can use [synthetic data](/definitions.md#synthetic-data-sd) to blend in the computation.

These requirements dictated the overall *query rewriting* architecture and many features, the most important of which, are detailed below.

## How does [Qrlew](https://qrlew.github.io/) work?

The [Qrlew](https://qrlew.github.io/) library, solves the problem of running a SQL query with [DP](/definitions.md#differential-privacy-dp) guarantees in three steps.
First the SQL query submitted by the [data practitioner](#data-practitioner) is parsed and converted into a [Relation](#qrlew-intermediate-representation), this [Relation](#qrlew-intermediate-representation) is an intermediate representation that is designed to ease the tracking of data types ranges or possible values, to ease the tracking of the [privacy unit](/definitions.md#datasets-and-privacy-units-pu) and to ease the rewriting into a DP *Relation*. Then, the rewriting into DP happens. Once the relation is rewritten into a DP one, it can be rendered as an SQL query string and submitted to the data store of the *data owner*. The output can then safely be shared with the *data practitioner*. This process is illustrated in {numref}`fig_qrlew_process`.

### Qrlew Intermediate Representation

As the SQL language is very rich and complex, simply parsing a query into an abstract syntax tree does not produce a convenient representation for our needs. Therefore, it is converted into a simpler normalized representation with properties well aligned with the requirements of Differential Privacy: the *Relation*. A *Relation* is a collection of rows adhering to a given *schema*. It is a recursively defined structure composed of:

* **Tables:** This is simply a data source from a database.
* **Maps:** A Map takes an input *Relation*, filters the rows and transform them one by one. The filtering conditions and row transforms are expressed with expressions similar to those of SQL. It acts as a `SELECT exprs FROM input WHERE expr LIMIT value` and therefore preserves the [privacy unit](/definitions.md#datasets-and-privacy-units-pu) ownership structure.
* **Reduces:** A *Reduce* takes an input *Relation* and aggregates some columns, possibly group by group. It acts as a `SELECT aggregates FROM input GROUP BY expr`. This is where the rewriting into DP will happen as described bellow.
* **Joins:** This *Relation* combines two input *Relations* as a `SELECT * FROM left JOIN right ON expr` would do it. The privacy properties are more complex to propagate in this case.

It may also be a static list of values or a set operation between two *Relations*, but those are less important for our uses.

```{figure} ./_static/relation.svg
:name: fig_relation

*Relation* (*Map*) associated to the query: `SELECT a, count(abs(10*a+b)) AS x FROM table_1 WHERE b>-0.1 AND a IN (1,2,3) GROUP BY a`. The arrows point to the inputs of each *Relation*. Note the propagation of the data type ranges.
```

This representation is central to [Qrlew](https://qrlew.github.io/); all the features described below are built upon it. A *Relation*, along with all the sub-*Relations* it depends on, will be called the *computation graph* or the *graph* of a *Relation*.

### Range Propagation

Most [DP](/definitions.md#differential-privacy-dp) mechanisms aggregating numbers require the knowledge of some bounds on the values (see {cite}`dwork2014algorithmic`).
Even if some bounds are known for some *Relations* like source **Tables**, it is not trivial to propagate these bounds through the steps of the computation.

To help with range propagation, [Qrlew](https://qrlew.github.io/) introduces two useful concepts:

* The concept of *$k$-Interval*, which are finite unions of at most $k$ closed intervals. A *$k$-Interval* can be noted:
$$I = \bigcup_{i=1}^{j\leq k}\left[a_i, b_i\right]$$
Note that the union of *$k$-Intervals* may not be a *$k$-Interval* as it may be the union of more than $k$ intervals.
Unions of many intervals can be simplified into their convex envelope interval, which are often sufficient bounds approximations for our use cases:
$$J = \bigcup_{i=1}^{j> k}\left[a_i, b_i\right] \subseteq \left[\min_i a_i, \max_i b_i\right]$$
* And the concept of *piecewise-monotonic-functions*[^pcmf], which are functions $f: \mathbb{R}^n \rightarrow \mathbb{R}$ whose domain can be partitioned in cartesian products of intervals: $P_j$ on which they are *coordinatewise-monotonic*.
The image of a cartesian product of $n$ *$k$-Intervals* by a *piecewise-monotonic-function* can be easily computed as a *$k$-Interval*.
Indeed, let $I$ be:
$$I = I_1\times I_2\times \ldots \times I_n = \bigcup_{\substack{1\leq i_1\leq k\\\ldots\\1\leq i_n\leq k}}\left[a_{i_1}, b_{i_1}\right]\times \ldots \times \left[a_{i_n}, b_{i_n}\right]$$
If $f$ is *piecewise-monotonic*, then one can show that on each partition $P_j$ where it is *coordinatewise-monotonic*, if we note:
$$I_j = I \cap P_j = \bigcup_{\substack{1\leq j_1\leq k\\\ldots\\1\leq j_n\leq k}}\left[a_{j_1}, b_{j_1}\right]\times \ldots \times \left[a_{j_n}, b_{j_n}\right]$$
$$f(I_j) = \bigcup_{\substack{1\leq i_1\leq k\\\ldots\\1\leq i_n\leq k}}\text{Conv}\left(f\left( \left\{a_{i_1}, b_{i_1}\right\}\times \ldots \times \left\{a_{i_n}, b_{i_n}\right\}\right)\right)$$
where $\text{Conv}\left(f\left( \left\{a_{i_1}, b_{i_1}\right\}\times \ldots \times \left\{a_{i_n}, b_{i_n}\right\}\right)\right)$ can be efficiently computed in $n$ steps, without testing all the $2^n$ combinations, thanks to the coordinatewise monotony of $f$ on $P_j$.
Then $f(I) = \bigcup_j f(I_j)$, of which we can derive the bounding: $f(I) \subseteq \text{Conv}\left(\bigcup_j f(I_j)\right)$ when the number of terms in the union exceeds $k$.

The notion of *$k$-Interval* is convenient for tracking value bounds as it can express natural patterns in SQL such as:
* `WHERE x>0 AND x<=1`, which translates into the implied $x\in \left[0, 1\right]$ ;
* `WHERE x IN (1,2,3)`, which is also easily expressed as a *k-Interval*: $x \in \left[1, 1\right] \cup \left[2, 2\right] \cup \left[3, 3\right]$

The idea of *piecewise-monotonic-function* is also very useful as in SQL many standard arithmetic operators (`+`, `-`, `\*`, `/`, `<`, `>`, `=`, `!=`, ...) and functions (`EXP`, `LOG`, `ABS`, `SIN`, `COS`, `LEAST`, `GREATEST`, ...) are trivially *piecewise-monotonic-function* (in one, two or many variables).

Most of the range propagation in [Qrlew](https://qrlew.github.io/) is based on these concepts. It enables a rather simple and efficient range propagation mechanism, leading to better utility / privacy tradeoffs.

### Privacy Unit Definition

Tables in a database rarely come properly formatted for privacy-preserving applications. Many rows in many tables may refer to the same individual, hence, *adding or removing an individual* means *adding or removing many rows*. To help the definition of the [privacy unit](/definitions.md#datasets-and-privacy-units-pu) [Qrlew](https://qrlew.github.io/) introduces a small [Privacy Unit (PU)](/definitions.md#datasets-and-privacy-units-pu) description language.
As exemplified in listing~\ref{lst:pe}, [PU](/definitions.md#datasets-and-privacy-units-pu) definition associates to each private table in a database a path defining the PID of each row. For a table containing the [PU](/definitions.md#datasets-and-privacy-units-pu) itself, like a `users` table for example, the PU definition will look like `("users",[],"id"),` where `id` is the name of a column identifying the user, like its name. If the database defines tables related to this tables, the way the tables are related should be specified following this scheme: $(\mathtt{tab}_1, path, \mathtt{pid})$ where $\mathtt{tab}_1$ is the name of the table for which the PID is defined, $\mathtt{pid}$ is the name of the column defining the PID in the table referred by $path$ and $path$ is a list of elements of the form $[(\mathtt{ref}_1, \mathtt{tab}_2, \mathtt{id}_2),\ldots, (\mathtt{ref}_{m-1}, \mathtt{tab}_m, \mathtt{id}_m)]$
where $\mathtt{ref}_{i-1}$ is a column in $\mathtt{tab}_{i-1}$ --- usually a foreign key --- referring to $\mathtt{tab}_i$ with a column of referred id $\mathtt{id}_i$ --- usually a primary key. Following the path of tables referring to one another, we end up with the table defining the PID (e.g. `users`).

This small [PU](/definitions.md#datasets-and-privacy-units-pu) description language allows for a variety of useful PID scenarii, beyond the simple, but restrictive *privacy per row*.

```{code-block} python
:caption: "Example of *privacy unit* definition for a database with three tables holding users, orders and items records. Each user is protected individually by designating their `id`s as PID. Orders are attached to a user through the foreign key: `user_id`. Items's ownership is defined the same way by specifying the lineage: `item -> order -> user`."

privacy_unit = [
    ("users",[],"id"),
    ("orders",[
    ("user_id","users","id")
    ],"id"),
    ("items",[
    ("order_id","orders","id"),
    ("user_id", "users", "id")
    ],"id")
]
```

### Rewriting

Rewriting in [Qrlew](https://qrlew.github.io/), refers to the process of altering the *computation graph* by substituting computation *sub-graphs* to *Relations* (see {numref}`fig_rewriting`) to alter the properties of the result. This substitution aims to achieve specific objectives, such as ensuring privacy through the incorporation of differentially private mechanisms. The rewriting process (see {numref}`fig_rewriting`) happens in two phases:
* a *rewriting rule allocation* phase, where each *Relation* in the *computation graph* gets allocated a *rewriting rule* (RR) compatible with its input and with the desired output property;
* a *rule application* phase, where each *Relation* is rewritten to a small *computation graph* implementing the logic of the rewriting and stitched together with the other rewritten *Relations*.

```{figure} ./_static/rewriting.svg
:name: fig_rewriting

The rewriting process happens in two phases: a *rewriting rule allocation* phase, where each node in the *computation graph* gets allocated a *rewriting rule* (RR) compatible with its input and with the desired output property; and a *rule application* phase, where each *Relation* is rewritten according to its allocated RR.
```

Before we describe these phases into more details, let's define the various properties we may want to guarantee on each *Relation* and the ones we need for the output.

#### Privacy Properties and Rewriting Rules

Each *Relation* can have one of the following properties:
* **Privacy Unit Preserving (PUP):** A *Relation* is PUP if each row is associated with a PU. In practice it will have a column containing the PID identifying the PU.
* **Differentially Private (DP):** A *Relation* will be DP if it implements a DP mechanism. A DP *Relation* can be safely  executed on private data and the result be published. Note that the *privacy loss* associated with the DP mechanism has to be accurately accounted for (see section~\ref{sec:privacy_analysis}).
* **Synthetic Data (SD):** In some contexts a *synthetic data* version of source tables is available. Any *Relation* derived from other SD or Public *Relations* is itself SD.
* **Public (Pub)** A relation derived from public tables is labeled as such and does not require any further protection to be disclosed.
* **Published (Pubd):** A relation is considered Published if its input relations are either Public, DP, in some cases SD, or Published themselves. It can be considered as Published but with some more care like the need to account for the privacy loss incurred by its DP ancestors.

These properties usually require some rewriting of the computation graph to be achieved. The requirements for a specific *Relation* to meet some property are embodied in what we call: *rewriting rules*.
A *rewriting rule* has input requirements, and an achievable output *property* that tells what *property* can be achieved by rewriting provided the input *property* requirements are fulfilled.
Each *Relation* can be assigned different *rewriting rules* depending on their nature: *Map*, *Reduce*, etc. and the way they are parametrized.

*Rewriting Rules* can be --- for instance --- PU propagation rules of the form:
* $\varnothing \rightarrow PUP$ for private *Tables* with a simple rewriting consisting in taking the definition of the privacy unit and computing the PID column.
* $PUP \rightarrow PUP$ for *Maps* (or for *Reduce* when the PID is in the `GROUP BY` part) with a rewriting consisting in propagating the PID column from the input to the output.
* $(PUP, PUP) \rightarrow PUP$ (or its variants with one published input) for *Join* and a rewriting consisting in adding the PID in the `ON` clause.

Another key *Rewriting Rules* is $PUP \rightarrow DP$ for *Reduces*, it simply means that if the parent of the *Relation* can be rewritten as PUP, then we can rewrite the relation to be DP by substituting DP aggregations to the original aggregations of the *Reduce*.

One easily see that by simply applying $PUP \rightarrow PUP$ and $PUP \rightarrow DP$ rules, one can propagate the privacy unit across the computation graph of a *Relation* and compute some DP aggregate such as a noisy sum or average.

#### Rewriting Rule Allocation

The first phase of the rewriting process consists in allocating one and only one rule to each *Relation*.
This is done in three steps illustrated in {numref}`fig_set_eliminate_select`:
* **Rule Setting:** We assign the set of potential rewriting rules to each *Relation* in a computation graph.
* **Rule Elimination:** Only feasible rewriting rules are preserved. A rewriting rule that would require a PUP input is only feasible if its input Relation has a feasible rule outputting a PUP *Relation*.
* **Rule Selection:** All feasible allocations of one rewriting rule per *Relation* are listed, a score depending on the desired ultimate output property is assigned to each allocation and the highest scoring allocation is selected. Then, a simple split $\left(\frac{\varepsilon}{n}, \frac{\delta}{n}\right)$ of the overall privacy budget $\left(\varepsilon, \delta\right)$ depending on the number of $PUP \rightarrow DP$ rules: $n$ is chosen.

In the computation graph, while each node's multiple rewriting rules might suggest a combinatorial explosion in the number of possible feasible allocations, this is mitigated in practice. The pruning of infeasible rules, dictated by the requirement for most relations to have a PUP input for a DP or PUP outcome, significantly reduces the complexity. Hence, despite the theoretical breadth of possibilities, the actual number of feasible paths remains manageable, avoiding substantial computational problems in practice.

```{figure} ./_static/set_eliminate_select.svg
:name: fig_set_eliminate_select

The rewriting happens in three steps: *Rule Setting* when we assign the set of potential rewriting rules to each *Relation* in a computation graph; *Rule Elimination*, when only feasible rewriting rules are preserved; and *Rule Selection*, when an actual allocation is selected.
```

#### Rule Application

Once the first phase of rule allocation is achieved, starts the second phase: *rule application*, as illustrated in {numref}`fig_rewriting`.
In the allocation phase, a *global rewriting scheme* was set in the form of an allocation satisfying a system of requirements; in the rewriting phase, each rewriting rule is applied *independently* for each *Relation*. This is possible because once a rewriting rule is applied to a *Relation*, the *Relation* is transformed into a computation graph of *Relations* whose ultimate inputs are compatible (same schema, i.e. same columns with same types, plus the new columns provided by the property achieved) with the inputs of the original *Relation* and the ultimate output is also compatible with the output of the original *Relation* so that rewritten *Relations* can be stitched together in a larger graph the same way the original *Relations* were connected: see {numref}`fig_rewriting`.

## Privacy Analysis

When rewriting, a user can require the output *Relation* to have the *Published* property. All *rewriting rules* with *Published* outputs require their inputs to be either *Public*, *DP*, *SD* or *Published* themselves. We assume synthetic data provided to the system are differentially private, so the privacy of the result depends on the way [Qrlew](https://qrlew.github.io/) rewrites *Reduces* into *DP* equivalent *Relations*.

All *rewriting rules* with *DP* outputs require the input of the *Reduce* to be *PUP* so we can assume a PID column clearly assign one and only one PU to each rows of the rewritten input. The *Reduce* is made DP by:
* Making sure the aggregate columns of the *Reduce* are computed with differentially private mechanisms.
* Making sure the grouping keys of the `GROUP BY` clause are either public or released through a differentially private mechanism.

### Protecting aggregation results

The protection of aggregation functions is carried out in two steps. Given that all currently supported aggregations (`COUNT`, `SUM`, `AVG`, `VARIANCE` `STDDEV`) can be reduced to sums, our focus will be on `SUM` aggregations, i.e. the computation of partial sums of a column for different groups: $j\in\{1,\ldots,m\}$, of rows.

Let the column be a vector of $N$ real numbers: $x = \left(x_1,\ldots, x_N\right)\in\mathbb{R}^N$. We note: $\pi_k = i \in \{1,\ldots,n\}$ the PID and $g_k = j \in \{1,\ldots,m\}$ the grouping key associated to $x_k$.
We want to compute all the sums:
$$S_j = \sum_{g_k = j} x_k$$
with some DP guarantees. To this end we:

* **Limit the contribution of each *privacy unit* to the sum**:
We represent the contribution of each PU: $i$, by a vector: $s_i$ whose components are the partial sums within each of the $m$ groups: $s_i = \left(s_{i,1},\ldots, s_{i,m}\right)$, where:
$$s_{i,j} = \sum_{\substack{\pi_k = i\\g_k = j}}x_k$$
The $s_i$'s $\ell^2$ norms are then clipped to $c$:
$$\overline{s_i} = \left(\overline{s_{i,j}}\right)_j = \left(\frac{s_{i,j}}{\max\left(1, \frac{\|s_i\|_2}{c}\right)}\right)_j$$
See the [whitepaper](#qrlew-white-paper) for more details.

* **Add gaussian noise to each group**:
The clipped contributions are summed and perturbed with gaussian noise $\nu = \left(\nu_1,\ldots \nu_m\right) \sim \mathcal{N}\left(0, \sigma^2I_m\right)$:
$$\widetilde{S_j} = \sum_{i=1}^n \overline{s_{i,j}} + \nu_j$$
With $\sigma^2={\frac {2\ln(1.25/\delta )\cdot c^{2}}{\varepsilon ^{2}}}$.
Note that the vector of sums has $\ell^2$ *Global Sensitivity* of $c$, so this is an application of the *Gaussian Mechanism* (see: theorem A.1. in {cite}`dwork2014algorithmic`) and the mechanism is $\varepsilon, \delta$-differentially private.

### Protecting grouping keys

When the grouping keys from a are derived from the data, they are not safe for publication.
Following {cite}`korolova2009releasing, wilson2019differentially`, we use a mechanism called *$\tau$-thresholding* to safely release these grouping keys.
Note that, thanks to *range propagation* (see section~\ref{sec:range_propagation}), some groups are already public and need no differentially private mechanism to be published.
Ultimately, the rewriting of: `SELECT sum(x) FROM table GROUP BY g WHERE g IN (1, 2, 3)` as a DP equivalent will not use *$\tau$-thresholding*, while `SELECT sum(x) FROM table GROUP BY g` will most certainly do if nothing more is known about `g` beforehand.

To summarize the various mechanisms used in [Qrlew](https://qrlew.github.io/) to date:
the rewriting of *Reduces* with $PUP \rightarrow DP$ rules requires the use of *gaussian mechanisms* and *$\tau$-thresholding* mechanisms;
then the DP mechanisms used in all the rewritings are aggregated by the [Qrlew](https://qrlew.github.io/) rewriter as a composed mechanism.
The overall privacy loss is aggregated in a RDP accountant {cite}`mironov2017renyi`.

## Comparison to other systems

There are a few existing open-source libraries for differential privacy.

Some libraries focus on deep learning and *DP-SGD* {cite}`abadi2016deep`, such as: *Opacus* {cite}`yousefpour2021opacus`, *Tensorflow Privacy* {cite}`TensorFlowPrivacy` or *Optax's DP-SGD* {cite}`deepmind2020jax`. [Qrlew](https://qrlew.github.io/) has a very different goal: analytics and SQL.

*GoogleDP* {cite}`GoogleDP` is a library implementing many differentially private mechanisms in various languages (C++, Go and Java).
*IBM's diffprivlib* {cite}`diffprivlib` is also a rich library implementing a wide variety of DP primitives in python and in particular many DP versions of classical machine learning algorithms. 
These libraries provide the bricks for experts to build DP algorithms. [Qrlew](https://qrlew.github.io/) has a very different approach, it is a high level tool designed to take queries written in SQL by a data practitioner with no expertise in privacy and to rewrite them into DP equivalent able to run on any SQL-enabled data store. [Qrlew](https://qrlew.github.io/) implemented very few DP mechanisms to date, but automated the whole process of rewriting a query, while these library offer a rich variety of DP mechanism, and give full control to the user to use them as they wish.

Google built several higher-level tools on top of {cite}`GoogleDP`.
*PrivacyOnBeam* {cite}`PrivacyOnBeam` is a framework to run DP jobs written in Apache Beam with its Go SDK.
*PipelineDP* {cite}`PipelineDP` is a framework that let analysts write Beam-like or Spark-like programs and have them run on Apache Spark or Apache Beam as back-end. It focuses on the Beam and Spark ecosystem, while [Qrlew](https://qrlew.github.io/) tries to provide an SQL interface to the analyst and runs on SQL-enabled back-ends (including Spark, a variety of data warehouses, and more traditional databases).
{cite}`ZetaSQL`, gives the user a way to write SQL-like queries and have them executed on tables using GoogleDB custom code, so it is not  compatible with any SQL data store and support relatively simple queries only.

*OpenDP* {cite}`OpenDP` is a powerful Rust library with a python bindings. It offers many possibilities of building complex DP computations by composing basic elements. Nonetheless, it require both expertise in privacy and to learn a new API to describe a query. Also, the computations are handled by the Rust core, so it does not integrate easily with existing data stores and may not scale well either.

*Tumult Analytics* {cite}`berghel2022tumult` shares many of the nice composable design of OpenDP, but runs on Apache Spark, making it a scalable alternative to OpenDP. Still, it require the learning of a specific API (close to that of Spark) and cannot leverage any SQL back-end.

*SmartNoise SQL* is a library that share some of the design choices of [Qrlew](https://qrlew.github.io/). An analyst can write SQL queries, but the scope of possible queries is relatively limited: no `JOIN`s, no sub-queries, no CTEs (`WITH`) that [Qrlew](https://qrlew.github.io/) supports. Also, it does not run the full computation in the DB so the integration with existing systems may not be straightforward.

Other systems such as *PINQ* {cite}`mcsherry2009privacy` and *Chorus* {cite}`johnson2020chorus` are prototypes that do not seem to be actively maintained. *Chorus* shares many of the design goals of [Qrlew](https://qrlew.github.io/), but requires post-processing outside of the DB, which can make the integration more complex on the data-owner side (as the computation happens in two distinct places).

Beyond that, [Qrlew](https://qrlew.github.io/) brings unique functionalities, such as:
* advanced automated range propagation;
* the possibility to automatically blend in synthetic data;
* advanced privacy unit definition capabilities across many related tables;
* the possibility for the non-expert to simply write standard SQL, but for the DP aware analyst to improve its utility by adding `WHERE x < b` or `WHERE x IN (1,2,3)` to give hints to the [Qrlew](https://qrlew.github.io/);
* all the compute happens in the DB.

This last point comes with some limitations (see section~\ref{sec:limitations}), but opens new possibilities like the delegation of the rewriting to a trusted third party. The data practitioner could simply write his desired query in SQL, send it to the rewriter that would keep track of the privacy losses and use [Qrlew](https://qrlew.github.io/) to rewrite the query, sign it, and send it back to the data practitioner that can then send the data-owner, who will check the signature certifying the DP properties of the rewritten query[^poc_server].

## Known Limitations

[Qrlew](https://qrlew.github.io/) still implements a limited number of DP mechanisms, it is still lacking basic functionalities such as: quantile estimation, exponential mechanisms.

[Qrlew](https://qrlew.github.io/) relies on the random number generator of the SQL engine used. It is usually not a cryptographic secure random number generator.

[Qrlew](https://qrlew.github.io/) uses the floating-point numbers of the host SQL engine, therefore it is liable to the vulnerabilities described in {cite}`casacuberta2022widespread`.

# Qrlew white paper

Read [Qrlew white paper](https://arxiv.org/pdf/2401.06273.pdf).

And cite us:
```bibtex
@misc{grislain2024qrlew,
  title={Qrlew: Rewriting SQL into Differentially Private SQL}, 
  author={Nicolas Grislain and Paul Roussel and Victoria de Sainte Agathe},
  year={2024},
  eprint={2401.06273},
  archivePrefix={arXiv},
  primaryClass={cs.DB}
  url={https://arxiv.org/pdf/2401.06273.pdf},
}
```
<!-- ```bibtex
@article{grislain2023qrlew,
  title={Qrlew: Rewriting SQL into Differentially Private SQL},
  author={Grislain, Nicolas and Roussel, Paul and de Sainte Agathe, Victoria},
  url={https://hal.science/hal-04350665},
  year={2023},
  month=Dec,
  keywords={SQL ; Privacy ; Differential Privacy ; Rewriting system ; Rust ; Privacy Enhancing Technology ; Privacy by Design},
  pdf={https://hal.science/hal-04350665/file/qrlew.pdf},
  hal_id={hal-04350665},
  hal_version={v1},
}
``` -->

# Definitions

Useful *definitions* can be found [there](/definitions.md).

# Bibliography

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

```{bibliography} ./qrlew.bib
```

[^pcmf]: Which is a shorthand name for what would be better called: *piecewise-coordinatewise-monotonic-functions*
[^poc_server]: A proof of concept is available at: <https://github.com/Qrlew/server>