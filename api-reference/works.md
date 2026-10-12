---
description: >-
  Add works one at a time or in bulk, then update or delete the ones your client added.
icon: book
---

# Add and update works

Works are research outputs: articles, datasets, software, and so on. Your client needs a token with the `/activities/update` scope that the researcher granted. Your client becomes the source of each work it adds, and only it can update or delete that work later.

{% hint style="info" %}
Everything on this page uses the **sandbox**: `pub.sandbox.orcid.org` and `api.sandbox.orcid.org`. In production, use the same paths on `pub.orcid.org` and `api.orcid.org`.
{% endhint %}

## What a work needs

A title, a type, and at least one external identifier. Without an identifier ORCID refuses the work (`400`, error `9023`). A `self` identifier, such as the work's DOI, is also how ORCID spots a duplicate.

## Add a work

`POST /v3.0/{orcid}/work` answers `201 Created` with an empty body. The new work's put-code is the last segment of the `Location` header.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i -X POST 'https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/work' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
    "title": { "title": { "value": "Toward a Unified Theory of High-Energy Metaphysics" } },
    "type": "journal-article",
    "publication-date": { "year": { "value": "2008" } },
    "external-ids": {
      "external-id": [
        { "external-id-type": "doi", "external-id-value": "10.5555/12345678", "external-id-relationship": "self" }
      ]
    }
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

orcid = "0000-0002-1825-0097"
work = {
    "title": {"title": {"value": "Toward a Unified Theory of High-Energy Metaphysics"}},
    "type": "journal-article",
    "publication-date": {"year": {"value": "2008"}},
    "external-ids": {
        "external-id": [
            {"external-id-type": "doi", "external-id-value": "10.5555/12345678", "external-id-relationship": "self"}
        ]
    },
}
response = requests.post(
    f"https://api.sandbox.orcid.org/v3.0/{orcid}/work",
    json=work,
    headers={"Authorization": "Bearer <ACCESS_TOKEN>", "Accept": "application/json"},
)
response.raise_for_status()
print("Created work", response.headers["Location"].rsplit("/", 1)[-1])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const orcid = "0000-0002-1825-0097";
const work = {
  title: { title: { value: "Toward a Unified Theory of High-Energy Metaphysics" } },
  type: "journal-article",
  "publication-date": { year: { value: "2008" } },
  "external-ids": {
    "external-id": [
      { "external-id-type": "doi", "external-id-value": "10.5555/12345678", "external-id-relationship": "self" },
    ],
  },
};
const response = await fetch(`https://api.sandbox.orcid.org/v3.0/${orcid}/work`, {
  method: "POST",
  headers: { Authorization: "Bearer <ACCESS_TOKEN>", "Content-Type": "application/json", Accept: "application/json" },
  body: JSON.stringify(work),
});
if (response.status !== 201) throw new Error(JSON.stringify(await response.json()));
console.log("Created work", response.headers.get("Location").split("/").pop());
```
{% endtab %}
{% endtabs %}

Adding the same `self` identifier again answers `409` (error `9021`), and the message names the put-code of the work your client already added.

## Add several works at once

`POST /v3.0/{orcid}/works` takes up to 100 works in a `bulk` list. It answers `200`, not `201`: the body has one element per work, either the created work with its put-code or the error that stopped it, so one bad work doesn't stop the others.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i -X POST 'https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/works' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
    "bulk": [
      { "work": { "title": { "title": { "value": "Psychoceramics" } }, "type": "book",
        "external-ids": { "external-id": [ { "external-id-type": "isbn", "external-id-value": "9780000000002", "external-id-relationship": "self" } ] } } },
      { "work": { "title": { "title": { "value": "On cracked pots" } }, "type": "journal-article",
        "external-ids": { "external-id": [ { "external-id-type": "doi", "external-id-value": "10.5555/87654321", "external-id-relationship": "self" } ] } } }
    ]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

orcid = "0000-0002-1825-0097"


def work(title, kind, id_type, id_value):
    return {"work": {
        "title": {"title": {"value": title}},
        "type": kind,
        "external-ids": {"external-id": [
            {"external-id-type": id_type, "external-id-value": id_value, "external-id-relationship": "self"}
        ]},
    }}


response = requests.post(
    f"https://api.sandbox.orcid.org/v3.0/{orcid}/works",
    json={"bulk": [
        work("Psychoceramics", "book", "isbn", "9780000000002"),
        work("On cracked pots", "journal-article", "doi", "10.5555/87654321"),
    ]},
    headers={"Authorization": "Bearer <ACCESS_TOKEN>", "Accept": "application/json"},
)
response.raise_for_status()
for element in response.json()["bulk"]:
    if "work" in element:
        print("Created", element["work"]["put-code"])
    else:
        print("Refused", element["error"]["error-code"])
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const orcid = "0000-0002-1825-0097";
const work = (title, type, idType, idValue) => ({
  work: {
    title: { title: { value: title } },
    type,
    "external-ids": {
      "external-id": [{ "external-id-type": idType, "external-id-value": idValue, "external-id-relationship": "self" }],
    },
  },
});
const response = await fetch(`https://api.sandbox.orcid.org/v3.0/${orcid}/works`, {
  method: "POST",
  headers: { Authorization: "Bearer <ACCESS_TOKEN>", "Content-Type": "application/json", Accept: "application/json" },
  body: JSON.stringify({
    bulk: [
      work("Psychoceramics", "book", "isbn", "9780000000002"),
      work("On cracked pots", "journal-article", "doi", "10.5555/87654321"),
    ],
  }),
});
if (!response.ok) throw new Error(JSON.stringify(await response.json()));
for (const element of (await response.json()).bulk) {
  console.log(element.work ? `Created ${element.work["put-code"]}` : `Refused ${element.error["error-code"]}`);
}
```
{% endtab %}
{% endtabs %}

## Update or delete a work

To change a work, send the whole work back with `PUT`, including its put-code. Fields you leave out are cleared. To remove it, send `DELETE` to the same URL; ORCID answers `204 No Content`.

{% tabs %}
{% tab title="cURL" %}
```bash
# Update: correct the title
curl -i -X PUT 'https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/work/1234567' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
    "put-code": 1234567,
    "title": { "title": { "value": "Toward a Unified Theory of High-Energy Metaphysics: Silly String Theory" } },
    "type": "journal-article",
    "external-ids": {
      "external-id": [
        { "external-id-type": "doi", "external-id-value": "10.5555/12345678", "external-id-relationship": "self" }
      ]
    }
  }'

# Delete
curl -i -X DELETE 'https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/work/1234567' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Accept: application/json'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

orcid = "0000-0002-1825-0097"
put_code = 1234567
url = f"https://api.sandbox.orcid.org/v3.0/{orcid}/work/{put_code}"
headers = {"Authorization": "Bearer <ACCESS_TOKEN>", "Accept": "application/json"}

# Update: send the whole work, with its put-code
work = {
    "put-code": put_code,
    "title": {"title": {"value": "Toward a Unified Theory of High-Energy Metaphysics: Silly String Theory"}},
    "type": "journal-article",
    "external-ids": {
        "external-id": [
            {"external-id-type": "doi", "external-id-value": "10.5555/12345678", "external-id-relationship": "self"}
        ]
    },
}
requests.put(url, json=work, headers=headers).raise_for_status()

# Delete
requests.delete(url, headers=headers).raise_for_status()
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const orcid = "0000-0002-1825-0097";
const putCode = 1234567;
const url = `https://api.sandbox.orcid.org/v3.0/${orcid}/work/${putCode}`;
const headers = { Authorization: "Bearer <ACCESS_TOKEN>", Accept: "application/json" };

// Update: send the whole work, with its put-code
const work = {
  "put-code": putCode,
  title: { title: { value: "Toward a Unified Theory of High-Energy Metaphysics: Silly String Theory" } },
  type: "journal-article",
  "external-ids": {
    "external-id": [
      { "external-id-type": "doi", "external-id-value": "10.5555/12345678", "external-id-relationship": "self" },
    ],
  },
};
const updated = await fetch(url, {
  method: "PUT",
  headers: { ...headers, "Content-Type": "application/json" },
  body: JSON.stringify(work),
});
if (!updated.ok) throw new Error(JSON.stringify(await updated.json()));

// Delete
const deleted = await fetch(url, { method: "DELETE", headers });
if (deleted.status !== 204) throw new Error(`Delete failed: ${deleted.status}`);
```
{% endtab %}
{% endtabs %}

Only the client that added a work can update or delete it. Any other client gets `403` (error `9010`).
