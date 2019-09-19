# SPIRE TPM Plugin

This repository contains agent and server plugins for [SPIRE](https://github.com/spiffe/spire) to allow TPM 2-based node attestation.

## Menu

- [Quick start](#quick-start)
- [How it Works](#how-it-works)
- [Building](#building)
- [Contributions](#contributions)
- [License](#license)
- [Code of Conduct](#code-of-conduct)
- [Security Vulnerability Reporting](#security-vulnerability-reporting)

## Quick Start

Before starting, create a running SPIRE deployment and add the following configuration to the agent and server:

### Agent Configuration

```hcl
NodeAttestor "tpm" {
	plugin_cmd = "/path/to/plugin_cmd"
	plugin_checksum = "sha256 of the plugin binary"
	plugin_data {
	}
}
```

### Server Configuration

```hcl
NodeAttestor "tpm" {
	plugin_cmd = "/path/to/plugin_cmd"
	plugin_checksum = "sha256 of the plugin binary"
	plugin_data {
		ca_path = "/opt/spire/.data/certs"
	}
}
```

| key | type | required | description | default |
|:----|:-----|:---------|:------------|:--------|
| ca_path  | string |   | the path to the CA directory | /opt/spire/.data/certs |

### Certificate Directory Configuration

For this plugin to work, you need to have the certificate for the CA that signed your TPM's EK certificate. Once you have the root and (if needed) intermediate certs, put the roots in `<ca_path>/RootCA/*.<cert>.{der,cer,crt}` and the intermediates in `<ca_path>/RootCA/<cert>.{der,cer,crt}`.

## How it Works

The plugin uses TPM credential activation as the method of attestation. The plugin operates as follows:

1. Agent generates AK (attestation key) using TPM
1. Agent sends the AK attestation parameters and EK certificate to the server
1. Server inspects EK certificate and checks if it is signed by any chain in the directory specified by `ca_path`
1. If the EK certificate is signed by one of the CAs, the server generates a credential activation challenge using
    1. The EK public key
    1. The AK attestation parameters
1. Server sends challenge to agent
1. Agent decrypts the challenge's secret 
1. Agent sends back decrypted secret
1. Server verifies that the decrypted secret is the same it used to build the challenge
1. Server creates a SPIFFE ID in the form of `spiffe://<trust_domain>/agent/tpm/<sha256sum_of_tpm_pubkey>`
1. All done!

For for info on how TPM attestation usually works and how this implementation differs, visit [TPM.md](TPM.md).

## Building

To build this plugin on Linux, run `make build`. Because of the dependency on [go-attestation](https://github.com/google/go-attestation), you must have `libtspi-dev` installed.

## Contributions

We :heart: contributions.

Have you had a good experience with this project? Why not share some love and contribute code, or just let us know about any issues you had with it?

We welcome issue reports [here](../../issues); be sure to choose the proper issue template for your issue, so that we can be sure you're providing the necessary information.

Before sending a [Pull Request](../../pulls), please make sure you read our
[Contribution Guidelines](https://github.com/bloomberg/.github/blob/master/CONTRIBUTING.md).

## License

Please read the [LICENSE](LICENSE) file.

## Code of Conduct

This project has adopted a [Code of Conduct](https://github.com/bloomberg/.github/blob/master/CODE_OF_CONDUCT.md).
If you have any concerns about the Code, or behavior which you have experienced in the project, please
contact us at opensource@bloomberg.net.

## Security Vulnerability Reporting

If you believe you have identified a security vulnerability in this project, please send email to the project
team at opensource@bloomberg.net, detailing the suspected issue and any methods you've found to reproduce it.

Please do NOT open an issue in the GitHub repository, as we'd prefer to keep vulnerability reports private until
we've had an opportunity to review and address them.
