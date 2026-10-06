---
description: >-
  Add an employment affiliation to a researcher's ORCID record with the Member
  API, from getting permission to handling errors.
icon: briefcase
---

# Add an employment

Use this endpoint to add an employment affiliation to a researcher's ORCID record. The researcher has to grant your member client permission first. Your client then becomes the item's **source**, so only your client can update or delete it later.

{% hint style="info" %}
Everything on this page uses the **sandbox**: `sandbox.orcid.org` for OAuth and `api.sandbox.orcid.org` for the API. You don't need to be an ORCID member to test there. In production, use the same paths on `orcid.org` and `api.orcid.org`.
{% endhint %}

## Before you start

New to the Member API? Read [Get started with the ORCID Member API](quickstart.md) first.

You need three things:

1. **Sandbox member API credentials**, a client ID and a client secret. [Request credentials](https://orcid.org/content/register-client-application).
2. A **redirect URI** registered on that client. ORCID sends the researcher back there after they grant access.
3. A **sandbox test researcher** to grant access. Create one at [sandbox.orcid.org/register](https://sandbox.orcid.org/register).

## Call the endpoint

{% stepper %}
{% step %}
### Ask the researcher for permission

Send the researcher to the authorization URL, asking for the `/activities/update` scope:

{% code overflow="wrap" %}
```
https://sandbox.orcid.org/oauth/authorize?client_id=APP-XXXXXXXXXXXXXXXX&response_type=code&scope=/activities/update&redirect_uri=https://your-app.example.org/orcid/callback
```
{% endcode %}

They sign in to ORCID and grant access. ORCID then sends them to your redirect URI with a short authorization code, for example `https://your-app.example.org/orcid/callback?code=Ab12Cd`.
{% endstep %}

{% step %}
### Exchange the code for an access token

{% code title="Token request" %}
```bash
curl -X POST 'https://sandbox.orcid.org/oauth/token' \
  -H 'Accept: application/json' \
  -d 'client_id=APP-XXXXXXXXXXXXXXXX' \
  -d 'client_secret=<CLIENT_SECRET>' \
  -d 'grant_type=authorization_code' \
  -d 'code=Ab12Cd' \
  -d 'redirect_uri=https://your-app.example.org/orcid/callback'
```
{% endcode %}

The response contains the token and the researcher's ORCID iD:

```json
{
  "access_token": "<ACCESS_TOKEN>",
  "token_type": "bearer",
  "refresh_token": "<REFRESH_TOKEN>",
  "expires_in": 631138518,
  "scope": "/activities/update",
  "name": "Josiah Carberry",
  "orcid": "0000-0002-1825-0097"
}
```

Store `access_token` and `orcid`. You need both for every call to this researcher's record.
{% endstep %}

{% step %}
### Add the employment

Send the employment as JSON to `/v3.0/{orcid}/employment` with three headers:

* `Authorization: Bearer <ACCESS_TOKEN>`
* `Content-Type: application/json`
* `Accept: application/json`. Without it, ORCID answers in XML.

{% tabs %}
{% tab title="cURL" %}
```bash
curl -i -X POST 'https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/employment' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
    "organization": {
      "name": "Boston University",
      "address": { "city": "Boston", "country": "US" },
      "disambiguated-organization": {
        "disambiguated-organization-identifier": "https://ror.org/05qwgg493",
        "disambiguation-source": "ROR"
      }
    }
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

orcid = "0000-0002-1825-0097"
employment = {
    "organization": {
        "name": "Boston University",
        "address": {"city": "Boston", "country": "US"},
        "disambiguated-organization": {
            "disambiguated-organization-identifier": "https://ror.org/05qwgg493",
            "disambiguation-source": "ROR",
        },
    }
}

response = requests.post(
    f"https://api.sandbox.orcid.org/v3.0/{orcid}/employment",
    json=employment,
    headers={
        "Authorization": "Bearer <ACCESS_TOKEN>",
        "Accept": "application/json",
    },
)
response.raise_for_status()
put_code = response.headers["Location"].rsplit("/", 1)[-1]
print("Created employment", put_code)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const orcid = "0000-0002-1825-0097";
const employment = {
  organization: {
    name: "Boston University",
    address: { city: "Boston", country: "US" },
    "disambiguated-organization": {
      "disambiguated-organization-identifier": "https://ror.org/05qwgg493",
      "disambiguation-source": "ROR",
    },
  },
};

const response = await fetch(
  `https://api.sandbox.orcid.org/v3.0/${orcid}/employment`,
  {
    method: "POST",
    headers: {
      Authorization: "Bearer <ACCESS_TOKEN>",
      "Content-Type": "application/json",
      Accept: "application/json",
    },
    body: JSON.stringify(employment),
  },
);
if (!response.ok) throw new Error(JSON.stringify(await response.json()));
const putCode = response.headers.get("Location").split("/").pop();
console.log("Created employment", putCode);
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### Keep the put-code

A successful call returns **`201 Created` with an empty body**. The new item's put-code is the last segment of the `Location` header:

```http
HTTP/1.1 201 Created
Location: https://api.sandbox.orcid.org/v3.0/0000-0002-1825-0097/employment/1234567
```

Store the put-code (`1234567` here) with your own record of the employment. You need it to update or delete the item later.
{% endstep %}
{% endstepper %}

## Try it from this page

The block below is generated from the OpenAPI spec. Use the example picker in the code panel to switch between bodies. Click **Test it** to send one to the sandbox: paste the `access_token` from step 2 and enter the researcher's iD as `orcid`. A `201` with a `Location` header means the employment was added.

{% openapi-operation spec="Orcid-POC" path="/v3.0/{orcid}/employment" method="post" %}
[OpenAPI Orcid-POC](https://raw.githubusercontent.com/cryptalith/temp-gitbook-poc/main/openapi/orcid-member-api.json)
{% endopenapi-operation %}

## More example bodies

<details>

<summary>Current position, with no end date</summary>

```json
{
  "department-name": "Department of Chemistry",
  "role-title": "Associate Professor",
  "start-date": { "year": { "value": "2021" }, "month": { "value": "09" }, "day": { "value": "01" } },
  "organization": {
    "name": "Boston University",
    "address": { "city": "Boston", "region": "MA", "country": "US" },
    "disambiguated-organization": {
      "disambiguated-organization-identifier": "https://ror.org/05qwgg493",
      "disambiguation-source": "ROR"
    }
  }
}
```

</details>

<details>

<summary>Past position, with dates and a URL</summary>

```json
{
  "department-name": "School of Public Health",
  "role-title": "Postdoctoral Researcher",
  "start-date": { "year": { "value": "2016" }, "month": { "value": "01" } },
  "end-date": { "year": { "value": "2019" }, "month": { "value": "12" }, "day": { "value": "31" } },
  "organization": {
    "name": "Boston University",
    "address": { "city": "Boston", "region": "MA", "country": "US" },
    "disambiguated-organization": {
      "disambiguated-organization-identifier": "https://ror.org/05qwgg493",
      "disambiguation-source": "ROR"
    }
  },
  "url": { "value": "https://www.bu.edu/sph/" }
}
```

</details>

<details>

<summary>With an external identifier (prevents duplicates)</summary>

```json
{
  "role-title": "Research Fellow",
  "start-date": { "year": { "value": "2023" } },
  "organization": {
    "name": "Boston University",
    "address": { "city": "Boston", "region": "MA", "country": "US" },
    "disambiguated-organization": {
      "disambiguated-organization-identifier": "https://ror.org/05qwgg493",
      "disambiguation-source": "ROR"
    }
  },
  "external-ids": {
    "external-id": [
      {
        "external-id-type": "other-id",
        "external-id-value": "BU-HR-000123",
        "external-id-url": { "value": "https://hr.example.org/staff/000123" },
        "external-id-relationship": "self"
      }
    ]
  }
}
```

If your client sends this body a second time, ORCID returns `409` (error 9021) instead of creating a duplicate.

</details>

## Rules worth knowing

| Field | Rule |
| --- | --- |
| `organization` | Required. It needs `name`, `address.city`, `address.country` and `disambiguated-organization`. |
| `disambiguated-organization` | The identifier and source must match an organization ORCID already knows. Prefer ROR, with the full ROR URL as the identifier. |
| `address.country` | ISO 3166-1 alpha-2 code in upper case, for example `US`. |
| `start-date`, `end-date` | Optional. The year is required, and the month and day can be left out. Send each part as a zero-padded string: `"09"`, not `9`. Years run from 1900 to 2100, and the start can't be after the end. Leave out `end-date` for a current position. |
| `url`, `external-id-url` | Objects, not strings: `{ "value": "https://..." }`. |
| `visibility` | On a claimed record, ORCID applies the researcher's own default visibility and ignores the value you send. |
| `put-code` | Never send it when you create. ORCID assigns it. |
| `external-ids` | Optional. A `self` identifier is what lets ORCID detect duplicates from your client. |

## Partial and invalid dates

Send a date as precisely as you know it:

| You know | Send |
| --- | --- |
| The year | `{ "year": { "value": "2023" } }` |
| The year and month | `{ "year": { "value": "2023" }, "month": { "value": "03" } }` |
| The full date | `{ "year": { "value": "2023" }, "month": { "value": "03" }, "day": { "value": "15" } }` |

A day without a month isn't allowed. ORCID also checks that a full date exists: `2023-02-30` returns `400` with error `9049`. To see that response, pick **Error: date that doesn't exist** in the example picker under the code panel.

## Errors

Errors come back as JSON with an `error-code` that says which rule failed:

```json
{
  "response-code": 400,
  "developer-message": "City and country are required for education/employment.",
  "user-message": "City and country are required for education/employment.",
  "error-code": 9060,
  "more-info": "https://members.orcid.org/api/resources/troubleshooting"
}
```

| HTTP | `error-code` | What happened | Fix |
| --- | --- | --- | --- |
| 400 | 9060 | City or country is missing from the organization address. | Send both. |
| 400 | 9045 | The organization's identifier and source don't match an organization ORCID knows. | Look the organization up on [ror.org](https://ror.org) and send its full ROR URL with source `ROR`. |
| 400 | 9024 | The body includes `put-code`. | Leave it out when creating. |
| 400 | 9055 | The end date is before the start date. | Fix the dates. |
| 400 | 9049 | A date that doesn't exist, such as 30 February. | Fix the date. |
| 400 | 9047 | Malformed JSON, an unknown field, an empty string, or a year outside 1900-2100. | Read the text after "Full validation error" in `developer-message`. |
| 401 | `invalid_token` | The token is missing, expired or revoked. | Get a new token (steps 1 and 2). |
| 401 | 9017 | The token was granted for a different ORCID iD. | Use the `orcid` from the token response in the URL. |
| 403 | 9038 | The token doesn't have `/activities/update`. | Ask for that scope in step 1. |
| 404 | 9016 | The ORCID iD doesn't exist. | Check the iD in the URL. |
| 409 | 9021 | Your client already added an employment with the same `self` external identifier. | Update the existing item with PUT. Its put-code is in `developer-message`. |
