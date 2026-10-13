---
description: >-
  Find ORCID iDs with Solr queries, with names and institutions, or as CSV.
icon: magnifying-glass
---

# Search the registry

Search finds records by name, identifier, affiliation and more, with [Solr query syntax](https://solr.apache.org/guide/solr/latest/query-guide/standard-query-parser.html). It works on the Public API without a token, and on the Member API with a `/read-public` token.

{% hint style="info" %}
Everything on this page uses the **sandbox**: `pub.sandbox.orcid.org`, or `api.sandbox.orcid.org` with a member client. In production, use the same paths on `pub.orcid.org` and `api.orcid.org`.
{% endhint %}

## Search

`GET /v3.0/search?q=...` answers with the number of matches and the iD of each.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i 'https://pub.sandbox.orcid.org/v3.0/search?q=family-name:Carberry' \
  -H 'Accept: application/json'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

response = requests.get(
    "https://pub.sandbox.orcid.org/v3.0/search",
    params={"q": "family-name:Carberry"},
    headers={"Accept": "application/json"},
)
response.raise_for_status()
found = response.json()
print(found["num-found"], [r["orcid-identifier"]["path"] for r in found["result"]])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const params = new URLSearchParams({ q: "family-name:Carberry" });
const response = await fetch(`https://pub.sandbox.orcid.org/v3.0/search?${params}`, {
  headers: { Accept: "application/json" },
});
if (!response.ok) throw new Error(`Search failed: ${response.status}`);
const found = await response.json();
console.log(found["num-found"], found.result.map((r) => r["orcid-identifier"].path));
```
{% endtab %}
{% endtabs %}

## Get names and institutions with the results

`/v3.0/expanded-search` takes the same query and answers with each match's names, emails and institutions, as far as they are public.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i 'https://pub.sandbox.orcid.org/v3.0/expanded-search?q=family-name:Carberry' \
  -H 'Accept: application/json'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

response = requests.get(
    "https://pub.sandbox.orcid.org/v3.0/expanded-search",
    params={"q": "family-name:Carberry"},
    headers={"Accept": "application/json"},
)
response.raise_for_status()
for match in response.json()["expanded-result"]:
    print(match["orcid-id"], match["given-names"], match["family-names"], match["institution-name"])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const params = new URLSearchParams({ q: "family-name:Carberry" });
const response = await fetch(`https://pub.sandbox.orcid.org/v3.0/expanded-search?${params}`, {
  headers: { Accept: "application/json" },
});
if (!response.ok) throw new Error(`Search failed: ${response.status}`);
for (const match of (await response.json())["expanded-result"]) {
  console.log(match["orcid-id"], match["given-names"], match["family-names"], match["institution-name"]);
}
```
{% endtab %}
{% endtabs %}

## Get the results as CSV

`/v3.0/csv-search` answers in CSV, with the fields you list in `fl`. Send `Accept: text/csv`.

```bash
curl -i 'https://pub.sandbox.orcid.org/v3.0/csv-search?q=family-name:Carberry&fl=orcid,given-names,family-name' \
  -H 'Accept: text/csv'
```

## Page through results

Use `start` (the offset of the first result) and `rows` (how many to return). `rows` can be at most 1,000. Without a token `start` can be at most 10,000. Beyond either limit ORCID answers `400`.
