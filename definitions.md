# Definitions

## Adjacent datasets

Datasets $D, D' \in \mathcal{D}$ are adjacent if they are equal up to the addition or removal of all entries sharing the same PID. Note that this is a slightly unusual and restricted definition of adjacency, suited to our practical needs. It is close to that used in the *user-level differential privacy* literature [^1] [^2] where one user can have many samples.

[^1]: Yuhan Liu, Ananda Theertha Suresh, Felix Xinnan X Yu, Sanjiv Kumar, and Michael Riley. Learning discrete distributions: user vs item-level privacy. Advances in Neural Information Processing Systems, 33:20965–20976, 2020.

[^2]: Royce J Wilson, Celia Yuxin Zhang, William Lam, Damien Desfontaines, Daniel Simmons-Marengo, and Bryant Gipson. Diﬀerentially private sql with bounded user contribution. arXiv preprint arXiv:1909.01917, 2019.

## Data Owner
The Data Owner is the person in charge of managing and protecting the datasets in a database.
The Data Owner can use Qrlew to rewrite the SQL queries of the [Data Practitioners](#data-practitioner) into Differentially Private equivalents,
run them on its behalf and account for privacy loss.

## Data Practitioner
The Data Practitioner is a data scientist, an analyst or any end user who wants to leverage sensitive data to carry out some sort of analysis.
They are interested in patterns that are true irrespective of a given individual but still want maximum utility when they query the datasets.

## Datasets and Privacy Units (PU)

In this documentation *datasets* refer to a collection of elements in some domain $\mathcal{X}$, labelled with an identifier $i\in \mathcal{I}$ identifying the entity whose privacy we want to protect. This entity will be called *Privacy Unit* (PU) and the identifier will be referred to as *Privacy ID* (PID). Let $\mathcal{D}$ be the set of datasets of arbitrary sizes with a privacy unit.

## Differential Privacy (DP)

Let $\mathcal{M}$ be an algorithm that takes a dataset as input and produces a randomized output. The algorithm $\mathcal{M}$ is said to satisfy $\varepsilon,\delta$-differential privacy if, for all pairs of adjacent datasets $D, D' \in \mathcal{D}$, and for all measurable sets $S$ in the range of $\mathcal{M}$:
$$
\Pr[\mathcal{M}(D) \in S] \leq e^{\varepsilon} \cdot \Pr[\mathcal{M}(D') \in S] + \delta
$$
For more background on DP, you can refer to: [differentialprivacy.org](https://differentialprivacy.org/resources/).

## Privacy Enhancing Technologies (PET)

As defined by the [United Nations PET Guide](https://unstats.un.org/bigdata/task-teams/privacy/guide/2023_UN%20PET%20Guide.pdf)
Privacy-enhancing technologies (PETs) are technologies
designed to safely process and share sensitive data.
There are two broad categories of PETs,
namely PETS for input and for output privacy.

**Input privacy** focuses on how one or multiple
parties can process data in a manner that guarantees
the data is not used outside of that strict context.

**Output privacy** focuses on modifying the results of a computation
such that the output data cannot be used to reverse
engineer the original inputs. By using these technologies
intelligently, safe data life cycles can be constructed,
enabling collaboration, trust and providing confidence to
data subjects.

## Synthetic Data (SD)

S
