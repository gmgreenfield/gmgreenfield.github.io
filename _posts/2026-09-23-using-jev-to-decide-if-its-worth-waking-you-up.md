---
layout: post
title: "Using Jev to Decide If a Problem Is Worth Waking You Up"
description: "An small introduction to Jev"
date: 2026-09-23 08:00:00 +0000
categories: [general]
tags: [ai, programming]
---
[Jev](https://docs.typesafe.ai/introduction){:target="_blank"} is an AI model designed to return decisions and probabilities directly to software. The interesting part is that TypeSafe trains it to make those probabilities reflect how often its answers are correct. If that holds up in practice, programs can use uncertainty to decide when a person needs to get involved.

Anyone who’s administered servers knows that twenty alerts can describe one failure. The disk fills up, the database stops accepting writes, and suddenly half your services are complaining.

You could use Jev to judge whether a new alert is explained by an incident you’re already handling.

```python
from typesafe_sdk import Choice, TypeSafeClient

with TypeSafeClient() as client:
    response = client.system_one(
        state=(
            "Active incident: db01 filesystem full; writes failing.\n"
            "New alert: checkout API returning HTTP 500.\n"
            "Checkout logs: INSERT failed: no space left on device."
        ),
        questions={
            "relationship": Choice(
                instructions="How does the new alert relate to the incident?",
                criteria={
                    "explained": "An expected consequence of the incident",
                    "separate": "Evidence points to an independent problem",
                    "uncertain": "Insufficient evidence to connect them",
                },
            )
        },
    )

answer = response.answers["relationship"]
if answer.probabilities["explained"] >= 0.95:
    print("Group under existing incident; retain the alert.")
else:
    print("Request separate review.")
```

Your code still controls the response. I’d test its judgments against actual incidents before letting it affect paging. The threshold here is illustrative; a reported probability isn’t proof.

END OF LINE
