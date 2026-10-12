---
description: >-
  Read a researcher's public record, or what they shared with your client.
icon: book-open
---

# Read a record

Anyone can read the public parts of an ORCID record through the Public API, without a token. Your member client can also read what a researcher shared with trusted parties, through the Member API, with a token that researcher granted.

{% hint style="info" %}
Everything on this page uses the **sandbox**: `pub.sandbox.orcid.org` and `api.sandbox.orcid.org`. In production, use the same paths on `pub.orcid.org` and `api.orcid.org`.
{% endhint %}

## Read the public record

Send `GET /v3.0/{orcid}/record` to the Public API. A token is optional: requests without one count against a daily quota per IP address, and requests with a `/read-public` token don't.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i 'https://pub.sandbox.orcid.org/v3.0/0000-0002-1825-0097/record' \
  -H 'Accept: application/json'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

orcid = "0000-0002-1825-0097"
response = requests.get(
    f"https://pub.sandbox.orcid.org/v3.0/{orcid}/record",
    headers={"Accept": "application/json"},
)
response.raise_for_status()
name = response.json()["person"]["name"]
print(name["given-names"]["value"], name["family-name"]["value"])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const orcid = "0000-0002-1825-0097";
const response = await fetch(`https://pub.sandbox.orcid.org/v3.0/${orcid}/record`, {
  headers: { Accept: "application/json" },
});
if (!response.ok) throw new Error(`Read failed: ${response.status}`);
const { name } = (await response.json()).person;
console.log(name["given-names"].value, name["family-name"].value);
```
{% endtab %}
{% endtabs %}

The record has two parts. `person` holds the names, biography, other names, keywords, links, emails, countries and identifiers. `activities-summary` holds a summary of each work, funding, affiliation, peer review and research resource, with its put-code. To read one section, use its own path, such as `/v3.0/{orcid}/works`. To read one item in full, use its put-code: `/v3.0/{orcid}/work/{putCode}`.

## Read what the researcher shared with you

Send the same request to the Member API with a token the researcher granted with the `/read-limited` scope. The answer also includes the items the researcher made visible to trusted parties, and every item your client added.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i 'https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/record' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Accept: application/json'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

orcid = "0000-0002-1825-0097"
response = requests.get(
    f"https://api.sandbox.orcid.org/v3.0/{orcid}/record",
    headers={"Authorization": "Bearer <ACCESS_TOKEN>", "Accept": "application/json"},
)
response.raise_for_status()
works = response.json()["activities-summary"]["works"]["group"]
print(len(works), "works")
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const orcid = "0000-0002-1825-0097";
const response = await fetch(`https://api.sandbox.orcid.org/v3.0/${orcid}/record`, {
  headers: { Authorization: "Bearer <ACCESS_TOKEN>", Accept: "application/json" },
});
if (!response.ok) throw new Error(`Read failed: ${response.status}`);
const works = (await response.json())["activities-summary"].works.group;
console.log(works.length, "works");
```
{% endtab %}
{% endtabs %}

A token from the client credentials grant with the `/read-public` scope also works on the Member API, for public items only.

## When a read fails

| HTTP | `error-code` | What happened |
| --- | --- | --- |
| 401 | | The token is invalid, expired or revoked. The body is `{"error": "invalid_token", ...}`. |
| 401 | 9017 | The token was granted for a different ORCID iD. |
| 404 | 9016 | No record has this iD. |
| 409 | 9018, 9044 | The record is locked or deactivated. |
| 301 | | The record was deprecated. `Location` is the same URL with the primary record's iD. |
