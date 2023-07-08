---
bookCollapseSection: true
---

# Function Apps

A brain dump covering content on function apps.

{{<section>}}

## Securing PaaS services when using a Linux based Consumption Plan

At the time of writing, `OutboundIPAddress` or `PossibleOutboundIPAddress` does not seem to be used for app service plans that use a Linux based consumption plan (`Y1`). Unfortuneatly, the consumption sku also does not support virtual network integration. This means that there doesn't appear to be a way to secure the network layer between a function app and a PaaS service that it consumes; such as a blob endpoint on a storage account (where the function is actually stored). Microsoft Learn documentation appears to contradict itself - [Find Outbound IP addresses](https://learn.microsoft.com/en-us/azure/azure-functions/ip-addresses?tabs=portal#find-outbound-ip-addresses) (see below for extract).

{{< expand >}}
### Function app outbound IP addresses
"Any outbound connection from a function, such as to a back-end database, uses one of the available outbound IP addresses as the origin IP address"

"The set of outboundIpAddresses is currently available to the function app. The set of possibleOutboundIpAddresses includes IP addresses that will be available only if the function app scales to other pricing tiers."

**_However_** there is also this on the same page:

"When a function app that runs on the Consumption plan or the Premium plan is scaled, a new range of outbound IP addresses may be assigned. When running on either of these plans, you can't rely on the reported outbound IP addresses to create a definitive allowlist. To be able to include all potential outbound addresses used during dynamic scaling, you'll need to add the entire data center to your allowlist."

So I have a sneaky feeling that `OutboundIPAddress` and `PossibleOutboundIPAddress` are not used at all for function apps; though it's not clear.

{{< /expand >}}
