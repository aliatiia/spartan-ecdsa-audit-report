# yAcademy Spartan ECDSA

Review Resources:

- [Spartan-ecdsa](https://github.com/personaelabs/spartan-ecdsa)

Auditors:

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

Findings are broken down into sections by their respective Impact:
 - Critical, High, Medium, Low Impact
     - These are findings that range from attacks that may cause loss of funds, proof malleability, or cause any unintended consequences/actions that are outside the scope of the requirements
 - Informational
     - Findings including Recommendations and best practices

---

## Critical Findings

None.

## High Findings

### 1. High - s is not constrained in ``eff_ecdsa.circom``

It is possible to submit `s = 0`, `Ux = pubX`, `Uy = pubY` or `s = 0`, `Ux = pubX`, `Uy = -pubY` and get back `(pubX, pubY)`, though this is not a valid signature.

#### Technical Details

Given check $\ s * T + U == pubKey\$ ,
```math
s * T + U == pubKey
```
```math
s = 0 ,  \forall  T  \in secp256k1
```
```math
s * T + U = 0 * T + U = O + U = U == pubKey
```
```math
or
```
```math
T = 0 , \forall s \in secp256k1
```
```math
s * T + U = s * 0 + U = O + U = U == pubKey 
```

where `U = (pubX, pubY)`, -U would work as well, where `-U = (pubX, -pubY)`. Here is a [POC](https://gist.github.com/igorline/c45c0fb84c943d1f641c82a20c02c21e#file-addr_membership-test-ts-L60-L66) to explain the same.

#### Impact

High. The missing constraints can be used to generate fake proof.

#### Recommendation

Add the constraints to the circuit and/or documentation

Reported by [Antonio Viggiano](https://github.com/aviggiano), [Igor Line](https://github.com/igorline), [Oba](https://github.com/obatirou)

### 2. High - Knowledge of any member signature allow to generate proof of membership

Knowing any valid signature by an account stored in the merkle tree, allows to generate relevant proof of being a member

#### Technical Details
There is no check on message supplied by the user thus user can submit any signature generated in the past with arbitrary message hashed

#### Impact
High. The missing constraints can be used to generate fake proof.

#### Recommendation
Add the constraints to the circuit and/or documentation

#### Developer Response

Reported by [Antonio Viggiano](https://github.com/aviggiano), [Igor Line](https://github.com/igorline), [Oba](https://github.com/obatirou)

### 3. High - Under constrained circuits compromising the soundness of the system

In the file [packages/circuits/eff_ecdsa_membership/secp256k1/mul.circom](), the signals `slo` & `shi` are assigned but not constrained.

#### Technical Details

```
    signal slo <-- s & (2  (128) - 1);
    signal shi <-- s >> 128;
```

#### Impact

High. Malicious provers can generate a fake proof in the case of an unsound system caused by underconstrained circuits.

#### Developer Response

> Adding the line `slo + shi * 2  128 === s;` would fix this, but it turns out that actually, that calculation of `k = (s + tQ) % q` doesn't have to be constrained at all (so the entire template K is unnecessary). Regardless, your discovery made me realize K is unnecessary, which results in solid constraint count reduction!

Reported by [nullity](https://github.com/nullity00)

### 4. High - X, Y pair may be an invalid point on the curve.

Circuits do not check whether the point $(x,y)$ is on the curve $E$.

#### Technical Details

The pair $\(x,y)\$ forms a group $G\$ of order $N\$ under $E(\mathbb{F}_p)/\mathcal{P}\$ where $E\$ represents an elliptic curve, $x, y < P\$ and $\mathbb{F}_p\$ denotes a finite field, and $\mathcal{P}\$ represents the prime order of the base point. But no check for whether $\(x,y)\$ in $G\$ is done. That is, the point $\(x,y)\$ is not being validated.

#### Impact

User may provide a public key (which is just a point $`(x,y)`$) that is not a valid point on the curve. This may leak the private key if the point is chosen from small order $N'$ of another curve $C'$

#### Recommendation

Validate the given point $(x,y)$ outside of the circuit.

Reported by [Rajesh](https://github.com/RajeshRk18)

#### Developer Response
## Medium Findings

None.

## Low Findings

### 1. Low - Unchecked edge case in complete addition
`Secp256k1AddComplete()` returns an incorrect value when `yP + yQ = 1`.

#### Technical Details
`zeroizeA.out` should be 0 when `P` and `Q` are different points, but when `xP != xQ` and `yP + yQ = 1` it would be 1.

In this case the output point would be the point at infinity instead of the actual sum.

#### Impact
Low. Secp256k1 arithmetics is incorrect in some edge cases.

#### Recommendation
Document the proof that $P$, $Q$ on the curve such that $yP + yQ = 1$ do not exist or are practically impossible to occurr.

If this can't be done, then add a `isYEqual` component as done for X and use `AND()` instead of `IsEqual()`
```
    component zeroizeA = AND();
    zeroizeA.in[0] <== isXEqual.out;
    zeroizeA.in[1] <== isYEqual.out;
```
There should be similar informational warnings to the client implementations for many edge cases like zero point, points at infinity, additions/multiplications with $\p\$ & $\-p\$

Reported by [Bahurum](https://github.com/bahurum), [0xnagu](https://github.com/thogiti)

## Informational Findings

### 1. Informational - Over-allocation of circom components

In mul.circom:Secp256k1Mul, the value accIncomplete and PComplete are over-allocated.

#### Technical Details

In [mul.circom:Secp256k1Mul](), the value `accIncomplete` and `PComplete` are over-allocated.
```
    component accIncomplete[bits];
    // ...
    component PComplete[bits-3]; 
```

#### Impact

Optimization.

#### Recommendation

Reduce the allocation of these component arrays to `accIncomplete[bits-p3]` and `PIncomplete[3]`.

Reported by [Antonio Viggiano](https://github.com/aviggiano), [Igor Line](https://github.com/igorline), [Oba](https://github.com/obatirou), [nullity](https://github.com/nullity00), [parsley](https://github.com/bbresearcher), 

### 2. Informational - Check if the input scalar is within the valid range

#### Technical Details

Add assertions and constraints to check for invalid inputs and edge cases

#### Impact

Informational.

#### Recommendation
Add a constraint to ensure that the input scalar is within the valid range of the Secp256k1 elliptic curve. You can do this by adding an assertion to check if the scalar is less than the curve's order.
```
// Add this line after the signal input scalar declaration
assert(scalar < 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141);
```

Reported by [0xnagu](https://github.com/thogiti)

### 3. Informational - Unused values

#### Technical Details

In `eff_ecdsa.circom`, the value `bits` is assigned but never read.

#### Impact

Informational.

#### Recommendation
Remove the unused value.

Reported by [Antonio Viggiano](https://github.com/aviggiano), [Igor Line](https://github.com/igorline), [Oba](https://github.com/obatirou), [garfam](https://github.com/gafram), [parsley](https://github.com/bbresearcher), [Bahurum](https://github.com/bahurum), [lwltea](https://github.com/lwltea)

### 4. Informational - No constraints on input signals

#### Technical Details

Likely due to a desire to reduce the number of constraints to a bare minimum, there are no constraints on input signals in any of the circuits. This could potentially cause issues for third party developers who use Spartan ECDSA.

#### Impact

Informational.

#### Recommendation

In order to keep the number of constraints to a minimum, simply document the absence of input signal constraints clearly and suggest that they be validated in the application code.

Reported by [whoismatthewmc](https://github.com/whoismatthewmc1)

### 5. Informational - Missing & Extra Imports in `eff_ecdsa.circom`

#### Technical Details

The `add.circom` import is missing in `eff_ecdsa.circom`. The `bitify.circom` is imported in eff_ecdsa.circom but not used.

#### Impact

Informational. This is not an issue as `add.circom` is imported in `mul.circom` which is in turn imported in `eff_ecdsa.circom`.

#### Recommendation

But recommendation is to explicitly import like `include "./secp256k1/add.circom";` & remove `bitify.circom` import.

Reported by [lwltea](https://github.com/lwltea), [Vincent Owen](https://github.com/makluganteng)

### 6. Informational - Constraints for add.cicom for values to be non-zero

In signal assignments containing division, the divisor needs to be constrained to be non-zero.

#### Technical Details
```
   │
31 │     lambda <-- dy / dx;
   │                     ^^ The divisor `dx` must be constrained to be non-zero.
```
#### Impact
Informational.

#### Recommendation
Do an additional check for non-zero values.

Reported by [Chen Wen Kang](https://github.com/cwkang1998), [Vincent Owen](https://github.com/makluganteng)

#### 7. Informational - More tests for the circuits
Additional tests are always good to have in order to cover more unexpected cases.

#### Technical Details
`eff_ecdsa.test.ts` and `eff_ecdsa_to_addr.test.ts` only have 1 positive tests.

#### Impact
Informational.

#### Recommendation
Adding more tests for the circuits.

Reported by [Chen Wen Kang](https://github.com/cwkang1998), [Vincent Owen](https://github.com/makluganteng)

## Final remarks

- The Spartan-Ecdsa circuits assume that the underlying hash function (Poseidon) is:
    - Collision-resistant
    - Resistant to differential, algebraic, and interpolation attacks
    - Behaves as a random oracle
- The Merkle tree used for membership proof is assumed to be secure against second-preimage attacks.
- Social engineering attacks are still a valid way to break the system.
ECDSA has several nonce based attacks. It is very important that the client side confirguration doesn't leak any nonce data or any app metadata that can reduce the security of guessing nonce for the ECDSA.
- Clarify the proper usage of each template, where assertions about the valuation of its inputs (pre-conditions) should be satisfied when calling the template.
- Write a detailed check list of things to do on the client side. This can help the developers not make standard mistakes of not validating the inputs and outputs and there by resulting into underconstrained related critical bugs.
- Overall, the code demonstrates good implementation of mathematical operations and basic functionality. However, it could benefit from more extensive documentation and additional testing and verification procedures.
