# Tracking Setup Playbook: GA4, GTM, Meta Pixel + Conversions API

The checklist I follow before spending a single dollar on ads. Platforms can only optimise on the data they receive, so **tracking comes first**.

## Contents

1. [Measurement plan](#1-measurement-plan)
2. [Google Tag Manager](#2-google-tag-manager)
3. [GA4](#3-ga4)
4. [Meta Pixel](#4-meta-pixel)
5. [Meta Conversions API (CAPI) and deduplication](#5-meta-conversions-api-capi-and-deduplication)
6. [Google Ads conversion tracking](#6-google-ads-conversion-tracking)
7. [UTM naming convention](#7-utm-naming-convention)
8. [Consent](#8-consent)
9. [QA checklist](#9-qa-checklist)

---

## 1. Measurement plan

Write this down before touching any tag:

| Business goal | Primary conversion | Secondary / micro conversions |
|---|---|---|
| E-commerce sales | `purchase` (with value + currency) | `add_to_cart`, `begin_checkout`, `view_item` |
| Lead generation | `generate_lead` (qualified form submit) | `form_start`, click-to-call, WhatsApp click |
| SaaS / app | `sign_up` or `start_trial` | demo request, pricing page view |

Rules:

- **One primary conversion per campaign objective.** Everything else is secondary, so it doesn't confuse bidding.
- Always send **value and currency** for purchases (and an estimated value for leads if you can), so you can bid on ROAS and POAS.

## 2. Google Tag Manager

- One GTM container per website, installed in `<head>` and right after `<body>`.
- Use a **data layer** for e-commerce events instead of scraping the page:

```js
window.dataLayer = window.dataLayer || [];
dataLayer.push({ ecommerce: null });           // clear previous ecommerce object
dataLayer.push({
  event: 'purchase',
  event_id: 'ORDER_10045',                     // reused for Meta deduplication
  ecommerce: {
    transaction_id: 'ORDER_10045',
    value: 129.90,
    currency: 'USD',
    items: [{ item_id: 'SKU_1', item_name: 'Product', price: 64.95, quantity: 2 }]
  }
});
```

- Naming convention: `GA4 - Event - purchase`, `Meta - Event - Purchase`, `GAds - Conversion - Purchase`.
- Use **Preview mode** for every change, and publish with a version note.

## 3. GA4

- Install the Google tag (GA4 configuration) on all pages through GTM.
- Mark `purchase` / `generate_lead` as **key events**.
- Turn on **enhanced measurement**, but switch off form interactions if they create noisy duplicate leads.
- Set data retention to **14 months**.
- Add **referral exclusions** for payment gateways (PayPal, Stripe, Klarna and similar), or they'll take credit for sales.
- Link GA4 to Google Ads and Search Console.

## 4. Meta Pixel

- Install the base code through GTM (or the native Shopify/WooCommerce integration) on all pages.
- Standard events: `ViewContent`, `AddToCart`, `InitiateCheckout`, `Purchase` (with `value` and `currency`), `Lead`.
- Verify the domain in **Business Manager → Brand safety → Domains**.
- In Events Manager, use **Aggregated Event Measurement** to rank events by priority, with Purchase or Lead at the top.

## 5. Meta Conversions API (CAPI) and deduplication

CAPI sends events from the server, so conversions are still recorded when browsers block the pixel (iOS, ad blockers, cookie limits).

Options, from easiest to most flexible:

1. **Native partner integration:** Shopify, WooCommerce, etc.
2. **CAPI Gateway** or Meta's server-side GTM template.
3. **Direct API integration** from the backend or CRM (best for offline and CRM lead events).

**Deduplication is mandatory** when the pixel and CAPI send the same event:

- Send the **same `event_name`** and the **same `event_id`** from both browser and server.
- Pass customer information (hashed email and phone, `fbp`, `fbc`, IP, user agent) to raise **Event Match Quality**. Aim for 6.0 or higher on Purchase.
- In Events Manager, check that deduplication shows events as "Deduplicated", not double-counted.

## 6. Google Ads conversion tracking

- Import GA4 key events **or** use the native Google Ads conversion tag. Pick one as primary for each action, not both.
- Turn on **Enhanced Conversions** (hashed first-party data).
- Set up the **conversion linker** tag in GTM.
- For lead gen, upload **offline conversions** (qualified or closed leads) using the GCLID, so bidding optimises for real sales.

## 7. UTM naming convention

Keep everything lowercase, with no spaces:

| Parameter | Meta | Google Ads |
|---|---|---|
| `utm_source` | `facebook` / `instagram` | `google` |
| `utm_medium` | `paid_social` | `cpc` |
| `utm_campaign` | `{{campaign.name}}` | `{campaignid}` |
| `utm_content` | `{{ad.name}}` | `{creative}` |
| `utm_term` | `{{adset.name}}` | `{keyword}` |

On Meta, put these in **URL parameters** at ad level, so dynamic values fill in automatically. On Google, use auto-tagging (GCLID) as well as UTMs.

## 8. Consent

- For EU/UK/EEA traffic, use a certified CMP with **Google Consent Mode v2** (`ad_storage`, `analytics_storage`, `ad_user_data`, `ad_personalization`).
- Only fire marketing tags after consent, or use Consent Mode so platforms can model the gaps.

## 9. QA checklist

- [ ] GTM Preview: each tag fires once, on the correct trigger
- [ ] GA4 DebugView: `purchase` arrives with value, currency, transaction_id and items
- [ ] Meta Events Manager → Test events: browser **and** server Purchase, deduplicated
- [ ] Event Match Quality ≥ 6 on Purchase or Lead
- [ ] Google Ads: conversion status "Recording conversions"; Enhanced Conversions active
- [ ] One test order or lead placed, and the numbers match across the shop/CRM, GA4, Meta and Google
- [ ] Payment gateways excluded as referrals
- [ ] UTMs present on every live ad
- [ ] Consent banner blocks or permits tags correctly

---

Built by **[Ali Raza](https://ali-raza-ppc.github.io)**, Performance Marketing Specialist · [LinkedIn](https://www.linkedin.com/in/ali-raza-ads-manager/)
