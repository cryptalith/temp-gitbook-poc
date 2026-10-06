---
description: >-
  This tutorial goes over editing information in the works section of an ORCID
  record. The work activity type is intended to link to the research outputs of
  the ORCID record holder.  This workflow can b
icon: globe
---

# Write, update, and delete works

### Overview

**Scopes:** `/activities/update` and `/read-limited`

**Method:** [3 step OAuth](https://github.com/ORCID/ORCID-Source/blob/master/orcid-api-web/README.md#authenticating-users-and-using-oauth--openid-connect)

**Endpoints:** `/work` and `/works`

**Sample XML files:**

* [reading the works section summary 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/read_samples/works-2.1.xml)
* [reading a basic work 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/read_samples/work-2.1.xml)
* [reading a detailed work item 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/read_samples/work-full-2.1.xml)
* [writing a work item with the minimal information 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/write_sample/work-simple-2.1.xml)
* [writing a work with the detailed information 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/write_sample/work-full-2.1.xml)
* [writing multiple works 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/write_sample/bulk-work-2.1.xml)
* [writing multiple works in json 2.1](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_2.1/samples/write_sample/bulk-work-2.1.json)
* [reading the works section summary 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/read_samples/works-3.0.xml)
* [reading a basic work 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/read_samples/work-3.0.xml)
* [reading a detailed work item 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/read_samples/work-full-3.0.xml)
* [writing a work item with the minimal information 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/write_samples/work-simple-3.0.xml)
* [writing a work with the detailed information 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/write_samples/work-full-3.0.xml)
* [writing multiple works 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/write_samples/bulk-work-3.0.xml)
* [writing multiple works in json 3.0](https://github.com/ORCID/orcid-model/blob/master/src/main/resources/record_3.0/samples/write_samples/bulk-work-3.0.json)

{% hint style="success" %}
SSL certificates are issued automatically once your domain is verified — there's nothing to install or renew yourself.
{% endhint %}

## Before you start

You'll need:

* Sandbox Member API credentials. Request them [here](https://info.orcid.org/register-a-client-application-sandbox-member-api/).
* Create a Sandbox ORCID ID. If you don't have one, create it [here](https://sandbox.orcid.org/).
* Access token with corresponding permission to the  `/activities/update` scope

## Post one new work

**Example request in curl**

```
curl -i -H 'Content-type: application/vnd.orcid+xml' -H 'Authorization: Bearer dd91868d-d29a-475e-9acb-bd3fdf2f43f4' -d '@[FILE-PATH]/work.xml' -X POST 'https://api.sandbox.orcid.org/v2.1/0000-0002-9227-8514/work'
```

## Steps

{% stepper %}
{% step %}
#### Add your domain in the dashboard

Go to **Project settings → Domains** and click **Add domain**. Enter the domain or subdomain you want to use:

```
docs.yourcompany.com
```

The dashboard will show you the DNS record values you need.
{% endstep %}

{% step %}
#### Configure your DNS

Add a CNAME record pointing your subdomain at the platform:

```
Type:  CNAME
Name:  docs
Value: sites.example-platform.com
TTL:   3600
```

For an apex domain (e.g. `yourcompany.com` without a subdomain), you'll need an ALIAS or ANAME record instead — not all DNS providers support these. Check your provider's docs.

{% hint style="info" %}
DNS changes can take up to 48 hours to propagate, but in practice most see them resolve within a few minutes.
{% endhint %}
{% endstep %}

{% step %}
#### Verify and activate

Return to **Project settings → Domains** and click **Verify**. Once verified, the platform will issue an SSL certificate automatically — usually within a couple of minutes.

When verification completes, your domain is live.
{% endstep %}
{% endstepper %}

## Multiple domains per project

You can attach multiple domains to a single project — useful for redirecting old URLs or supporting multiple regions:

```yaml
domains:
  primary: docs.yourcompany.com
  redirects:
    - from: help.yourcompany.com
      to: docs.yourcompany.com
      status: 301
```

## Troubleshooting

<details>

<summary>Verification keeps failing</summary>

Make sure there are no conflicting DNS records for the same subdomain. An existing A record will block the CNAME.

Also check that the value of your CNAME exactly matches the one shown in the dashboard — typos in domain names are very common.

</details>

<details>

<summary>SSL certificate not provisioning</summary>

SSL is issued after DNS verification succeeds. If it hasn't appeared after 30 minutes:

1. Confirm your domain still verifies in the dashboard
2. Check that your DNS provider isn't using CAA records that block our certificate authority
3. Click **Re-verify** to trigger a fresh attempt

</details>

<details>

<summary>Browser shows "Your connection is not private"</summary>

This usually means the certificate hasn't been provisioned yet, or your browser cached an old certificate. Wait a few minutes, then try in an incognito window.

</details>

<details>

<summary>The domain works but redirects break</summary>

Redirect chains often get cached aggressively. Test in an incognito window or with `curl -I` to see what's actually being served, ignoring local cache.

```bash
curl -I https://docs.yourcompany.com
```

</details>

## Removing a domain

To remove a domain, go to **Project settings → Domains**, click the menu next to the domain, and select **Remove**.

{% hint style="warning" %}
Removing a domain stops serving traffic immediately. If you've shared this URL externally, set up a redirect first to avoid breaking inbound links.
{% endhint %}
