---
aip: (this is determined by the AIP Manager, leave it empty when drafting)
title: Prover Service for Aptos Keyless Accounts
author: Rex Fernando, Alin Tomescu
discussions-to (*optional): <a url pointing to the official discussion thread>
Status: Draft
last-call-end-date (*optional): <mm/dd/yyyy the last date to leave feedbacks and reviews>
type: <Standard (Core, Networking, Interface, Application, Framework) | Informational | Process>
created: <mm/dd/yyyy>
updated (*optional): <mm/dd/yyyy>
requires (*optional): <AIP number(s)>
---

# AIP-X - Prover Service for Aptos Keyless Accounts

## Summary

This AIP is an extension of [AIP-61](https://github.com/aptos-foundation/AIPs/blob/main/aips/aip-61.md): Keyless Accounts, which allow users to have a wallet which is tied to an OpenID account, and to authenticate with the blockchain via their OIDC provider. As summarized in AIP-61, “your blockchain account = your OIDC account”. OpenID authenticates users based on personally-identifying information, e.g. an email address or a twitter handle. We want to guarantee:

- The OpenID provider does not learn which wallets are linked to which users.
- The validators (and other outside observers) also cannot learn the link between wallets and OpenID users.

In order to do this, we are using a zero-knowledge proof which each user must provide to validators to authenticate transactions. Generating such a proof must be done each time a user logs in, and then each time the user's ephemeral public key[^spec], and is computationally intensive. To allow for users to login quickly and on low-powered hardware, we plan to offload this proof computation to a proving service.

### Goals

1. Enable Keyless account users to login quickly and without friction. 
2. Respect users' privacy as much as possible.
3. Build a service that is relatively inexpensive host.
5. Protect against bugs in the zero-knowledge system.

### Out of Scope

We are not trying to solve the issue of privacy between the user and the prover service. That is, we are *allowing* the prover service to learn the user's private information, including:
* The user's OIDC handle. For example, if logging in with Google, the prover service will learn the user's email.
* The user's privacy-preserving pepper[^spec].

The fact that the prover service learns this information induces privacy and centralization risks. These risks are discussed [below](##Risks and Drawbacks).


## Motivation

The motivation of this AIP follows directly from the motivation of [AIP-61](https://github.com/aptos-foundation/AIPs/blob/main/aips/aip-61.md). The purpose of Aptos Keyless is to greatly reduce friction in onboarding and key management for users. Specifically, the Keyless Proving Service will allow for the most computationally intensive step during login to be offloaded to a powerful cloud VM instead of being done locally, thus greatly improving the user experience of Aptos Keyless.
 


## Impact

The direct impact of this AIP will be on users of Aptos Keyless accounts. The impact will be twofold:
* Users will have a much faster login experience than they would if we were generating proofs client-side. From preliminary benchmarks, generating proofs in-browser takes such a long time that is completely unusable. (i.e., > 25 seconds to generate the proof.)
* On the other hand, users's private information will be sent to the prover service.

## Alternative solutions

The most obvious alternative is requiring the user to generate a proof client-side. As discussed above, this solution is untenable, at least with the current ZKP system that is implemented in Aptos Keyless.

## Specification

 > How will we solve the problem? Describe in detail precisely how this proposal should be implemented. Include proposed design principles that should be followed in implementing this feature. Make the proposal specific enough to allow others to build upon it and perhaps even derive competing implementations.

...


### API:

The prover service is accessed via the following endpoint:

* https://prover.keyless.devnet.aptoslabs.com/v0/prove

The prover service API consists of the required format for requests along with the format which responses take. Both input and response formats are defined by the following:

* [Prover service request and response structs](https://github.com/aptos-labs/prover-service/blob/master/src/api.rs)


## Reference Implementation

The main code repository for the prover service is linked here:

* [Prover service code](https://github.com/aptos-labs/prover-service)

## Testing (Optional)

 > - What is the testing plan? (other than load testing, all tests should be part of the implementation details and won’t need to be called out)
 > - When can we expect the results?
 > - What are the test results and are they what we expected? If not, explain the gap.

...

## Risks and Drawbacks

 > - Express here the potential negative ramifications of taking on this proposal. What are the hazards?
 > - Any backwards compatibility issues we should be aware of?
 > - If there are issues, how can we mitigate or resolve them?

- If we don’t sufficiently optimize the circuit and prover code, the prover service could be cost-prohibitive to scale.
    - Solution: robust benchmarks of prover, understanding of cost involved in running the service
- Currently, the prover service learns all
-

## Future Potential

 > Think through the evolution of this proposal well into the future. How do you see this playing out? What would this proposal result in one year? In five years?

...

## Timeline

### Suggested implementation timeline

 > Describe how long you expect the implementation effort to take, perhaps splitting it up into stages or milestones.

...

### Suggested developer platform support timeline

 > Describe the plan to have SDK, API, CLI, Indexer support for this feature is applicable. 

...

### Suggested deployment timeline

 > Indicate a future release version as a *rough* estimate for when the community should expect to see this deployed on our three networks (e.g., release 1.7).
 > You are responsible for updating this AIP with a better estimate, if any, after the AIP passes the gatekeeper’s design review.
 >
 > - On devnet?
 > - On testnet?
 > - On mainnet?

...

## Security Considerations

 > - Does this result in a change of security assumptions or our threat model?
 > - Any potential scams? What are the mitigation strategies?
 > - Any security implications/considerations?
 > - Any security design docs or auditing materials that can be shared?

* DDOS-ing prover service

## Open Questions 

The research team at Aptos plans to spend considerable time on how to mitigate the privacy and centralization compromises encompassed in this AIP. Specifically, we plan to work on the following questions:

* Can we design a new ZKP with sufficient performance to allow for client-side proving, and thus eliminate the prover service altogether?
* If not, can we design a prover service which is "blind", i.e., it does not learn any sensitive information about users?

## References

[^spec]: https://github.com/rex1fernando/AIPs/blob/main/aips/aip-61.md#specification