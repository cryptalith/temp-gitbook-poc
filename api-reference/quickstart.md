---
description: Credentials, OAuth and your first call, in the sandbox.
icon: rocket
---

# Get started with the ORCID Member API

The ORCID Member API lets ORCID member organizations write to researchers' ORCID records, and read what researchers have shared with them, with each researcher's permission. These docs cover the **employment** section of the v3.0 API.

## Sandbox and production

| | Sandbox | Production |
| --- | --- | --- |
| Use it for | Building and testing, with test records | Real researchers' records |
| OAuth (sign-in and tokens) | `https://sandbox.orcid.org/oauth/...` | `https://orcid.org/oauth/...` |
| API | `https://api.sandbox.orcid.org/v3.0` | `https://api.orcid.org/v3.0` |
| Credentials | Anyone can [request them](https://orcid.org/content/register-client-application); no membership needed | ORCID members only |

Build against the sandbox first. Moving to production means changing the hosts and the credentials, nothing else.

## How access works

Your client can only change a researcher's record after the researcher grants permission. ORCID uses 3-legged OAuth for this:

1. Your app sends the researcher to ORCID's authorize page, naming the scopes it needs.
2. The researcher signs in to ORCID and grants access. ORCID sends them back to your redirect URI with a one-time code.
3. Your server exchanges the code for an access token. The response also tells you the researcher's ORCID iD.
4. You call the API with that token, on that iD's record.

Store the token with the iD. `expires_in` in the token response says how long the token lasts.

The scopes for employment are:

| Scope | Lets your client |
| --- | --- |
| `/activities/update` | Add employments and other activities, and update or delete the ones your client added. |
| `/read-limited` | Read items the researcher shared with trusted parties, not only public ones. |

## Two rules for every call

- Send `Accept: application/json`. Without it, ORCID answers in XML, errors included.
- Use HTTPS. A plain HTTP call returns `400` with error `9012`.

## Your first call

Add an employment to your sandbox test researcher's record. [Add an employment](add-employment.md) walks through the whole call, from asking for permission to reading the put-code from the response, with examples you can send from the page.

To change or remove that employment later, keep its put-code. You need it to update or delete the item; both calls are in the Reference section.
