---
pcx_content_type: how-to
title: Tenant nameservers
weight: 5
---

# Tenant-level nameservers

Tenant-level custom nameservers can be part of any domain, even if the domain does not exist as a zone within any account in Cloudflare. These nameservers are organized in different sets (`ns_set`) and can be applied and used across different accounts, as long as the accounts are part of the [tenant](/tenant/).

## Configuration conditions
For this configuration to be possible, a few conditions apply:

1. If the domain that is used for the tenant custom nameservers does not exist within the same account, the tenant owner must create the A/AAAA records on the configured nameserver names (e.g. ns1.example.org) at the authoritative DNS provider.
2. Tenant owners can create up to five different tenant nameserver sets. Each nameserver set must have between two and five different nameserver names (ns_name) and each name cannot belong to more than one set. For example, if ns1.example.com is part of ns_set 1 it cannot be part of ns_set 2 or vice versa.
3. Subdomain or Reverse zones can use tenant-level custom nameservers as long as they use a different nameserver set (ns_set) than their parent or child.

{{<Aside>}}
Tenant owners that want to [use their own IP prefix](/byoip/) for the tenant-level custom nameserver should contact their account team.
{{</Aside>}}

## Enable tenant nameservers on a zone
If you are an account owner and your account is part of a tenant that has custom nameservers, you can enable the tenant-level custom nameservers on your zones by using a PUT command and specifying `ns_type` and `ns_set`.

``` bash
curl -- request PUT "https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/custom_ns" \
  -- header "X-Auth-Email: <EMAIL>" \
  -- header "X-Auth-Key: <KEY>" \
  -- header "Content-Type: application/json" \
  -- data '{
    "enabled":true,
    "ns_type":"tenant",
    "ns_set": <SET>
  }'
```

{{<Aside>}}
If the parameter `ns_type` is omitted, the default type `account` will be assigned.
If the parameter `ns_set` is omitted, the default set `1` will be assigned.
{{</Aside>}}

## Make tenant nameservers default for new zones

To make this custom nameserver the default for all new zones added to your account from now on, use a [PUT command](/api/operations/accounts-update-account) on the account with the following parameters.

``` bash
curl -- request PUT “https://api.cloudflare.com/client/v4/accounts/{account_id}” \
  -- header "X-Auth-Email: <EMAIL>" \
  -- header "X-Auth-Key: <KEY>" \
  -- header "Content-Type: application/json" \
  -- data '{
    "zone_onboarding": {
        “Nameservers”: “tenant_custom_ns”
    }
}'
```

{{<Aside>}}
Note that you cannot use this setting and have [`use_account_custom_ns_by_default` set to `true`](/dns/additional-options/custom-nameservers/account-level-nameservers/#add-account-nameservers) at the same time. Only one of these options can be set per account.
{{</Aside>}}