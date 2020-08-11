# Wraith Signatures

## Overview

These signatures are designed for [wraith][1] but the expressions themselves can be considered general purpose. They provide the capability to match against file names and content, finding possible secrets within code and other digital hiding placs.

There are 3 types of signatures:

- Simple Signatures
- Pattern Signatures
- Whitelist Signatures

### Simple Signatures

These match part of a path, filename (or both).

```yaml
SimpleSignatures:
  -
    part: PartExtension # match a file extension
    match: .ovpn # extension to match against
    description: 'OpenVPN client configuration file' # description of the file type
    signatureid: '9f136035162f4ee6be29e8d55aff9ef4' # MD5 of the expression, used as a relatively unique identifer
    enable: 1 # enable the signature, 0 will disable it
    entropy: 0 # the Shannon Entropy value for a signature to match. 0 disables entropy and all matches reported. A good value for passwords is at least 4
    match-level: 3 # expression confidence level, the lower the number the less false positives but also the higher chance that something may be missed.
```
### Pattern Signatures

These are expressions that match content *within* a file. The regex must conform to [re2][2] project standards. There are a few differences between this and PCRE. Negative and positive lookaheads and lookbehinds are not currently supported by re2 and should not be included in any regex's submitted here. The expressions can be tested at [regex101][4]. There is talk of switching to a different library but in reality the development time would be better spent with implementing other language specific detection methods and using expressions as a fallback method.

```yaml
PatternSignatures:
    -
        part: PartContent # Content matching
        match: '(?i)(\bAKIA[0-9A-Z]{16}\b)' # expression to look for
        description: 'Contains AWS Access Key ID' # description of what we are looking for
        signatureid: '8e8a22f07f655b280cdfb06a9a929823' # MD5 of the expression, used as a relatively unique identifer
        enable: 1 # enable the signature, 0 will disable it
        entropy: 3.5 # the Shannon Entropy value for a signature to match. 0 disables entropy and all matches reported. A good value for passwords is at least 4
        match-level: 0 # expression confidence level, the lower the number the less false positives but also the higher chance that something may be missed.
        test_file: sample_aws_keys # file with sample text in many common languages to test against.
```
### Whitelist Signatures

These are expressions run against results identified as possible passwords. If the results match a regex here, they are considered acceptable and are not flagged as a password. This feature exists as a way to reduce false positives through whitelisting.

```yaml
SafeFunctionSignatures:
    -
        part: PartContent # Content matching
        match: '(?i)\$\{\w.+?\}' # expression to look for
        description: 'Environment Variable' # description of what we are looking for
        signatureid: '08cb551ea0104c198fbe91d6f75b6e87' # MD5 of the expression, used as a relatively unique identifer
        enable: 1 # enable the signature, 0 will disable it
        match-level: 3 # expression confidence level, the lower the number the less false positives but also the higher chance that something may be missed.
```
## Signature File Format

The signature file is well structured yaml and can be linted and verrified using [yamllint.com][3]. 

At the start of a session during the init phase wraith will read the signature file(s) and parse them to ensure there are no validation errors before entering the discovery phase. Any errors in the YAML file will be printed to the console and the program will exit.

```yaml
SimpleSignatures:
    -
        part: PartExtension
        match: .ovpn
        description: 'OpenVPN client configuration file'
        ruleid: '9f136035162f4ee6be29e8d55aff9ef4'
        enable: 1
        entropy: 0
        match-level: 3
```

## Installation and Usage

Wraith is bundled with the latest available signatures when each version is released. Updating to new signatures can be accomplished by cloning this repo or downloading a stable release and installing the signatures manually to the default location `$HOME/.wraith/signatures/` or anywhere else. You can also download the latest release of wraith which contains the current stable default signatures that can be copied to the default location or called with the `--signature-file` flag.

## Signature Files

There may be various signature files found within the signatures directory at any given time. The most stable will always be found within the *default_signatures.yml* file. Other files will have additional documentation as to their specific purpose and level of stability. Any signature files in the pattern of `test_<signature>` should be considered unstable and untested.

[1]: https://github.com/mattyjones/wraith
[2]: https://github.com/google/re2/wiki/Syntax
[3]: http://www.yamllint.com
[4]: http://regex101.com
