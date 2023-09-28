# yAcademy Spartan ECDSA

**Review Resources:**

- [Spartan-ecdsa](https://github.com/personaelabs/spartan-ecdsa)

**Auditors:**

- [0xnagu](https://github.com/thogiti)
- [Antonio Viggiano](https://github.com/aviggiano)
- [Bahurum](https://github.com/bahurum)
- [Chen Wen Kang](https://github.com/cwkang1998)
- [garfam](https://github.com/gafram)
- [Igor Line](https://github.com/igorline)
- [lwltea](https://github.com/lwltea)
- [nullity](https://github.com/nullity00)
- [Oba](https://github.com/obatirou)
- [parsley](https://github.com/bbresearcher)
- [Rajesh](https://github.com/RajeshRk18)
- [Vincent Owen](https://github.com/makluganteng)
- [whoismatthewmc](https://github.com/whoismatthewmc1)

## Table of Contents

- [Review Summary]()
- [Scope]()
- [Findings Explanation]()
- [Critical Findings]()
- [High Findings]()
- [Medium Findings]()
- [Low Findings]()
- [Informational Findings]()

## Review Summary

**Spartan-ecdsa**

Spartan-ecdsa is the fastest open-source method to verify ECDSA (secp256k1) signatures in zero-knowledge. It can prove ECDSA group membership 10 times faster than efficient-zk-ecdsa, personaelabs' previous implementation of fast ECDSA signature proving. [Spartan](https://github.com/microsoft/Spartan) is a high-speed zero-knowledge proof system which does not require trusted setup. Spartan uses ``curve25519-dalek`` for arithmetic over ``ristretto255`` whereas spartan-ecdsa uses the ``secp256k1 curve``.

The circuits of the Spartan-ecdsa repo were reviewed over 15 days. The code review was performed by 13 auditors between June 19, 2023 and July 5, 2023. The repository was under active development during the review, but the review was limited to the latest commit at the start of the review. This was commit [3386b30d9b](https://github.com/personaelabs/spartan-ecdsa/tree/3386b30d9b5b62d8a60735cbeab42bfe42e80429) for the Spartan-ecdsa repo.

## Scope

The scope of the review consisted of the following circuits at the specific commit:

- eff_ecdsa.circom
- tree.circom
- add.circom
- double.circom
- mul.circom
- poseidon.circom
- pubkey_membership.circom

After the findings were presented to the Spartan-ecdsa team, fixes were made and included in several PRs.

This review is a code review to identify potential vulnerabilities in the code. The reviewers did not investigate security practices or operational security and assumed that privileged accounts could be trusted. The reviewers did not evaluate the security of the code relative to a standard or specification. The review may not have identified all potential attack vectors or areas of vulnerability.

yAcademy and the auditors make no warranties regarding the security of the code and do not warrant that the code is free from defects. yAcademy and the auditors do not represent nor imply to third parties that the code has been audited nor that the code is free from defects. By deploying or using the code, Spartan-ecdsa and users of the circuits agree to use the code at their own risk.


Code Evaluation Matrix
---

| Category                 | Mark    | Description |
| ------------------------ | ------- | ----------- |
| Access Control           | N/A | Spartan-ecdsa is a permissionless protocol, and as such no access control is required |
| Mathematics              | Good | Sage scripts were created to assess the security of some parameters used in the algorithms |
| Complexity               | High | Spartan-ecdsa achieves performance improvements over standard ECDSA algorithms by changing how the signature verification mechanism is implemented. As such, it integrates most of its intricacies, consequently enhancing the overall complexity of the codebase |
| Libraries                | Average | Spartan-ecdsa uses well known libraries, such as circomlib, whenever possible. However, due to the focus on efficiency, some library circuits were reimplemented from scratch |
| Decentralization         | Good | Spartan-ecdsa is a permissionless protocol |
| Cryptography           | Good    | Spartan-ecdsa makes use of `secp256k1 curve` which provides a security level of `128 bits`. It makes use of the Poseidon hash function known for its efficiency, simplicity, and resistance against various cryptanalytic attacks. However, it's essential to note that cryptographic algorithms and functions are always subject to ongoing analysis, and new attacks or weaknesses may be discovered in the future. |
| Code stability           | Average    | The code was reviewed at a specific commit. The code did not changed during the review. However, due to its focus on efficiency, it is likely to change with the addition of features or updates, or to achieve further performance gains. |
| Documentation            | Low | Spartan-ecdsa documentation comprises some [blog posts](https://personaelabs.org/posts/spartan-ecdsa/) from Personae Labs, the Github [README](https://github.com/personaelabs/spartan-ecdsa/blob/main/README.md) documentation, and some reference materials from [Filecoin](https://spec.filecoin.io/#section-algorithms.crypto.poseidon), and [Neptune](https://github.com/lurk-lab/neptune). It is recommended to aggregate the resources necessary to the understanding of the protocol under a single repository |
| Monitoring               | N/A | The protocol is intended to be implemented by a dapp, which will be responsible for the monitoring |
| Testing and verification | Low | The protocol contains only a few tests for the circuits. During audit, we developed [circom-mutator](https://github.com/aviggiano/circom-mutator), a mutation testing tool designed to help find missing spots in the test coverage of circom projects. The circom-mutator tool found that several edge cases that were not tested by the project. It is recommended to add more tests to increase the test coverage |

## Findings Explanation

Findings are broken down into sections by their respective impact:
 - Critical, High, Medium, Low impact
     - These are findings that range from attacks that may cause loss of funds, proof malleability, or cause any unintended consequences/actions that are outside the scope of the requirements
 - Informational
     - Findings including recommendations and best practices

---

## Critical Findings

None.

## High Findings

### 1. High - s is not constrained in eff_ecdsa.circom

It is possible to submit s = 0, Ux = pubX, Uy = pubY or s = 0, Ux = pubX, Uy = -pubY and get back (pubX, pubY), though this is not a valid signature.

Technical Details
Given check

```math
s * T + U == pubKey Q_a
For s = 0 and \(\forall\)
```


where U = (pubX, pubY), -U would work as well, where -U = (pubX, -pubY)

POC: https://gist.github.com/igorline/c45c0fb84c943d1f641c82a20c02c21e#file-addr_membership-test-ts-L60-L66

Impact
High. The missing constraints can be used to generate fake proof.

Recommendation
Add the constraints to the circuit and/or documentation

Developer Response

## Medium Findings

None.

## Low Findings

### 1. Low - TODO_Title

TODO

#### Technical Details

TODO

#### Impact

Low. TODO_reasoning.

#### Recommendation

TODO

#### Developer Response



### 2. Low - TODO_Title

TODO

#### Technical Details

TODO

#### Impact

Low. TODO_reasoning.

#### Recommendation

TODO

#### Developer Response



## Gas Savings Findings

### 1. Gas - TODO_Title

TODO

#### Technical Details

TODO

#### Impact

Gas savings.

#### Recommendation

TODO

## Informational Findings

### 1. Informational - TODO_Title

TODO

#### Technical Details

TODO

#### Impact

Informational.

#### Recommendation

TODO

## Final remarks

TODO
