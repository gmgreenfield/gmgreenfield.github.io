---
layout: post
title: "Using the macOS Secure Enclave for SSH Keys"
date: 2026-11-11
categories: [macos, ssh, security]
---

Sure, your SSH private keys are protected in `~/.ssh` via file permissions and passphrases, but a process running under your account may be able to copy them.

If you'd like to enhance the way your keys are stored, consider using the secure enclave offered by MacOS. Rather than storing it as a plain file, it's generated in hardware and can't be exported. Then, to access it OpenSSH uses a local reference to request signatures without handling the private key itself.

Additionally, with biometric protection each signing operation also would also require activation by Touch ID.

## Creating the key

Of course to take advantage of this benefit, we need a Mac with Secure Enclave support and a macOS installation that provides the relevant `sc_auth` commands. Before attempting this please check `man sc_auth`.

Create a new identity:

```sh
sc_auth create-ctk-identity \
    -l "ssh-main" \
    -k p-256-ne \
    -t bio
```

The options specify a non-exportable NIST P-256 key and require biometric authentication when it is used.

Verify that the identity exists:

```sh
sc_auth list-ctk-identities
```

## Making the key available to SSH

OpenSSH needs a local reference to the hardware-backed identity. Generate it using Apple's CryptoTokenKit provider:

```sh
mkdir -p ~/.ssh
cd ~/.ssh

ssh-keygen -w /usr/lib/ssh-keychain.dylib -K -N ""
```

This creates a key reference and its corresponding public key. The reference is not the private key; the actual signing operation remains inside the Secure Enclave.

The resulting files are typically named `id_ecdsa_sk_rk` and `id_ecdsa_sk_rk.pub`.

## Configuring OpenSSH

Add the identity and provider to `~/.ssh/config`:

```sshconfig
Host my-server
    HostName server.example.com
    User username
    IdentityFile ~/.ssh/id_ecdsa_sk_rk
    SecurityKeyProvider /usr/lib/ssh-keychain.dylib
    IdentitiesOnly yes
```

Replace the example hostname, username, and identity filename with your own values.

Retrieve the public key:

```sh
cat ~/.ssh/id_ecdsa_sk_rk.pub
```

Add its contents to the remote account's `~/.ssh/authorized_keys`, as you would with a conventional SSH key.

Connect normally:

```sh
ssh my-server
```

The system should request Touch ID before using the key.

## Security and recovery

The main advantage is that stealing files from `~/.ssh` is no longer sufficient to obtain the private key. Signing happens inside the Secure Enclave, and biometric protection adds a requirement for local user approval.

This does not make a compromised Mac harmless. Malicious software may still request a signing operation and attempt to persuade you to approve it. Hardware-backed storage prevents key extraction; it does not prevent every form of unauthorized key use.

Secure Enclave keys cannot be exported or backed up. They are tied to the hardware that created them.

Keep a second SSH key on another device or hardware authenticator and register its public key with your servers. Otherwise, losing the Mac could mean losing access to every system that trusts the key.

## References

- [Apple: Protecting Keys with the Secure Enclave](https://developer.apple.com/documentation/Security/protecting-keys-with-the-secure-enclave)
- [OpenSSH: `ssh-keygen`](https://man.openbsd.org/ssh-keygen.1)
- [OpenSSH: `ssh_config`](https://man.openbsd.org/ssh_config)
- [Serge Gatezh: Protecting SSH Keys with macOS Secure Enclave](https://gatezh.com/posts/macos-secure-enclave-ssh-keys/)
