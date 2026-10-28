---
layout: post
title: "When a Certificate Authority Becomes the Punchline"
description: "The eMudhra key compromise exposed how a broken certificate can outlive the browsers that stopped trusting it."
date: 2026-10-28 18:00:00 +0000
categories: [general]
tags: [security, macos, certificates]
---

A certificate authority is supposed to be one of the quiet, boring parts of the internet. Your browser trusts a list of roots; those roots vouch for intermediates; and the intermediates help prove that a site is really the site it claims to be. The whole arrangement depends on private keys remaining private and on clients actually learning when trust should end.

The eMudhra incident made that arrangement look like a joke. According to the [incident report](https://emudhra.we-have-a-problem.com/), the private keys for eight intermediate certificates were generated incorrectly, in a way that allowed them to be recovered from the certificates themselves. The keys could be used to impersonate websites. The reporter demonstrated possession of the keys and notified root program teams; browsers and several major platforms moved to block the affected intermediates.

The especially absurd part for Mac developers was the gap between “Apple has addressed this” and “the command-line tools I use are safe.” Apple’s networking stack had blocked the intermediates, but Apple’s bundled command-line Git still trusted a root in `/etc/ssl/cert.pem`. On my Mac, `git ls-remote` against the incident’s test endpoint succeeded. That was not an abstract warning: a Git HTTPS connection could accept a certificate chained through the compromised authority.

I backed up the system CA bundle and removed the affected `emSign ECC Root CA - G3` entry. Afterward, Git rejected the same test connection with an issuer error. I left the separate AAA Certificate Services root in place because removing it can break unrelated sites. This is a narrow workaround, and a macOS update may replace the bundle, so it is worth checking again after updates.

A CA whose certificate keys can be reconstructed has failed at the most basic job its name promises. But the broader punchline is systemic: revocation only protects people when every client that relies on the trust store gets the memo. Browsers did; my Git did not. “HTTPS” is only as trustworthy as the certificate validation path underneath it.

[Incident details and checks](https://emudhra.we-have-a-problem.com/) · [Hacker News discussion](https://news.ycombinator.com/item?id=50271640)
