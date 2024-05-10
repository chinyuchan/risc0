# Trusted Setup Security

RISC Zero has run a trusted setup ceremony for our Groth16 prover/verifier. This ceremony secures our STARK Verify circuit so we can publish compact receipts for our general purpose zkVM to limited-memory environments like blockchains.

You don't need to take our word for it that this is secure, though! This document will walk you through how to verify the security of our ceremony for yourself. This can also be used to aid in the detection of attacks related to our ceremony, e.g. it may help detect fraudulent actors publishing something they claim to be the RISC Zero STARK Verifier but which is actually malicious code.

## Overview

There are multiple steps to verifying the security of a trusted setup ceremony. Kobi Gurkan gave [a good list of what is required][kobi-bad-ceremony-list] in the form of a list of ways a malicious project could pretend to run a ceremony while secretly retaining a backdoor. Following this model, we will cover the following steps to verify our ceremony is secure:

1. Verifying the circuit we are securing is what we intend to secure and does not include security holes.
2. Verifying the transcript corresponds to this circuit
3. Verifying contributors' attestations match the transcript
4. Verifying the setup ceremony does not include security holes

## The Circuit

The circuit we are securing is the RISC Zero STARK Verify circuit, which is open source and [available on GitHub][stark-verify-circom] (along with [a short library][risc0-circom-library] it depends on).

To ensure that the circuit itself does not have security holes, we have used a mixture of good software engineering practices, internal security reviews, and external audits. Stay tuned for detailed reports of our audit results!

## The Transcript Matches the Circuit

Our ceremony transcript is included in the `zkey` published on ceremony.pse.dev in the "Download ZKey" tab of the [RISC Zero STARK-to-SNARK Prover page][pse-risc0-ceremony]. You can verify it matches the circuit using Circom and snarkjs:

1. Install [Circom][install-circom] and [snarkjs][snarkjs].
1. Download the [`stark_verify.circom`][stark-verify-circom] and [`risc0.circom`][risc0-circom-library] source files.
1. Download the Powers of Tau (`ptau`) file we used for our ceremony; we used the Hermez rollup with `2^23` powers of tau, which is linked in [the snarkjs readme][snarkjs] or available directly [here][powers-of-tau-hez-23] (mirrored [here][powers-of-tau-hez-23-our-mirror]).
1. Generate the `r1cs` file from the `circom` source files, by running `circom stark_verify.circom --r1cs` in the directory where you downloaded these files. This should generate the same `r1cs` file as listed in our [p0tion configuration][p0tion-config] (available directly [here][r1cs-file]). The SHA-256 hash of this file (i.e. as computed by `shasum -a 256`) is `84d3c34b7c0eb55ad1b16b24f75e0b9de307f7b74089ea4a20a998390ee24178`.
1. Prepare JavaScript to use a large amount of memory: Its default settings generally are insufficient to verify this circuit. On my system, I needed to run `export NODE_OPTIONS="--max-old-space-size=32768"`.
1. Use snarkjs to verify that the transcript matches this circuit and powers of tau, by running `snarkjs zkey verify stark_verify.r1cs powersOfTau28_hez_final_23.ptau stark_verify_final.zkey`. You should see a list of contribution hashes (attestations) followed by the message `snarkJS: ZKey Ok!`.

## Contributor Attestations Match the Transcript

Each contributor authorized our ceremony to publish an attestation to their GitHub account. (In a Gist entitled "`risc-zero-stark-to-snark-prover_attestation.log`") These attestations match the attestations published in the transcript. This proves which GitHub user made each contribution to the ceremony. You can verify attestations by looking them up both in the transcript and on GitHub:

In the transcript (generated by `snarkjs` in the previous step), you will see that each contribution is preceded by the phrase `contribution #[number] [username]-[number]`. That username is the GitHub username.

On GitHub, you can run a search to find an attestation Gist on gist.github.com. For example, if the username is `somecontributor`, then you could search for `filename:"risc-zero-stark-to-snark-prover_attestation.log" user:somecontributor`.

To verify an attestation, confirm that the hash in the attestation in the transcript matches the hash in the GitHub Gist for that same user.

If you are looking for your own contribution, you can also go to gist.github.com and navigate to your Gist named "`risc-zero-stark-to-snark-prover_attestation.log`" (which will be linked at the top if you don't make Gists for other reasons; otherwise you can look for it under "View your gists" or with the search function as described in the previous paragraph). You can also find your contribution the same way as for any other user (i.e. by searching the transcript for your username).

Important Note: Contributors can remove their attestations from GitHub at any time. They can also edit their attestations (although in this case the edit history will be visible). _Only the original version of the attestation can be valid; an edited version cannot be a valid attestation_. Note that that if any malicious contributors were able to participate in the ceremony, it does not damage the security of the ceremony, but it _does_ mean that they can pretend to have a bad attestation by editing or deleting their Gist. Therefore, a contribution with no attestation provides no security to the ceremony, but does not necessarily mean anything is wrong, either.

Please exercise good judgment about whether a missing or edited attestation represents:
1. A malicious contributor
1. Someone just cleaning up old Gists
1. A problem in the ceremony

## Ceremony Security

We used the open-source tools [p0tion] and [DefinitelySetup] to run our ceremony, and our ceremony was coordinated by the [PSE] team. This gave us tools that had been battle-tested by prior ceremonies, and moreover, by using tools written by an external team, we put substantial limits on our own ability to maliciously manipulate the ceremony software.



[DefinitelySetup]: https://github.com/privacy-scaling-explorations/DefinitelySetup
[install-circom]: https://docs.circom.io/getting-started/installation/
[kobi-bad-ceremony-list]: https://twitter.com/kobigurk/status/1782502969453494530
[p0tion]: https://github.com/privacy-scaling-explorations/p0tion
[p0tion-config]: https://github.com/risc0/risc0/blob/d4e427283027c28b38b8eda1562e8e0e68d1b0e2/compact_proof/groth16/p0tionConfig.json
[powers-of-tau-hez-23]: https://storage.googleapis.com/zkevm/ptau/powersOfTau28_hez_final_23.ptau
[powers-of-tau-hez-23-our-mirror]: https://risc0-artifacts.s3.us-west-2.amazonaws.com/tsc/2024-04-04/powersOfTau28_hez_final_23.ptau
[PSE]: https://pse.dev
[pse-risc0-ceremony]: https://ceremony.pse.dev/projects/RISC%20Zero%20STARK-to-SNARK%20Prover
[r1cs-file]: https://risc0-artifacts.s3.us-west-2.amazonaws.com/tsc/2024-04-04/stark_verify.r1cs
[risc0-circom-library]: https://github.com/risc0/risc0/blob/d4e427283027c28b38b8eda1562e8e0e68d1b0e2/compact_proof/groth16/risc0.circom
[snarkjs]: https://github.com/iden3/snarkjs
[stark-verify-circom]: https://github.com/risc0/risc0/blob/d4e427283027c28b38b8eda1562e8e0e68d1b0e2/compact_proof/groth16/stark_verify.circom