+++
title = "How to point a Route 53 apex domain to GitHub Pages"
description = "A step-by-step guide to use a root domain bought in AWS with GitHub Pages."
date = 2026-04-30T00:00:00Z
showAuthor = false
showReadingTime = true
showTaxonomies = true
featureimage = "img/articles/github-pages-route53.svg"
tags = ["github-pages", "aws", "dns"]
categories = ["howto"]
+++

GitHub Pages supports custom domains. If your domain is managed in AWS Route 53, you can point the root domain, for example `opeshm.net`, directly to your GitHub Pages site instead of using only a subdomain such as `blog.opeshm.net`.

The setup happens in two places: the DNS records in Route 53 and the GitHub Pages settings in the repository.

## Before you start

You need:

- A domain managed in Route 53, for example `opeshm.net`.
- A repository with GitHub Pages enabled.
- Access to `Settings > Pages` in the repository.

The examples use `opeshm.net` as the root domain. Replace it with your own domain.

## 1. Configure the A records in Route 53

For a root domain, also called an apex domain, you do not use a `CNAME`. GitHub Pages publishes fixed IP addresses, and the root domain must point to them with `A` records.

In AWS:

1. Open the AWS console.
2. Go to `Route 53`.
3. Open `Hosted zones`.
4. Select the hosted zone for the domain, for example `opeshm.net`.
5. Create a record for the root domain by leaving `Record name` empty.
6. Use type `A`.
7. Add these values:

```text
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Depending on the Route 53 interface, you can add them as one `A` record with multiple values or as equivalent records.

## 2. Configure the www subdomain

It is usually worth making `www.yourdomain.com` work too.

In Route 53, create this record:

```text
Record name: www
Type: CNAME
Value: your-user.github.io
```

Replace `your-user.github.io` with the GitHub Pages domain for your account or organization.

With this, `www.opeshm.net` will point to the same site as the root domain.

## 3. Configure the domain in GitHub Pages

After creating the DNS records, GitHub needs to know that it should accept requests for the domain.

In GitHub:

1. Open the repository.
2. Go to `Settings`.
3. Open `Pages`.
4. In `Custom domain`, enter the root domain, for example `opeshm.net`.
5. Save the change.

GitHub will check the DNS records. Validation can take from a few minutes to a few hours, depending on DNS propagation.

If the site is published with a GitHub Actions workflow, it is also useful to keep the custom domain as a `CNAME` file in the generated site. In Hugo, a simple way is to create `static/CNAME` with this content:

```text
opeshm.net
```

Hugo will copy it to `public/CNAME` during the build.

## 4. Enable HTTPS

When GitHub validates the domain, go back to `Settings > Pages` and enable `Enforce HTTPS`.

GitHub will automatically issue a free TLS certificate for the domain. The option can take a few minutes to become available while the certificate is generated.

## 5. Decide what to do with the old subdomain

If you previously used `blog.opeshm.net`, you have two options:

- Keep it if you still want it to resolve somewhere.
- Delete it from Route 53 if you no longer want to use it.

Keep in mind that a GitHub Pages repository only has one primary custom domain configured in `Settings > Pages`. If you want to keep the subdomain, you will usually use it as a redirect or for a separate site.

## Final DNS setup

A typical setup looks like this:

| Name | Type | Value |
| --- | --- | --- |
| `opeshm.net` | `A` | `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` |
| `www.opeshm.net` | `CNAME` | `your-user.github.io` |
| `blog.opeshm.net` | `CNAME` | Only if you still need it |

## Check the result

After DNS propagation finishes, check that the domain resolves correctly:

```bash
dig opeshm.net A
dig www.opeshm.net CNAME
```

Then open the site in the browser:

```text
https://opeshm.net
```

If GitHub Pages serves the site and `Enforce HTTPS` is enabled, the migration from the subdomain to the root domain is complete.
