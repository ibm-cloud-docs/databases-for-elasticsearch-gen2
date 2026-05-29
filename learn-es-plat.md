---
copyright:
  years: 2026
lastupdated: "2026-05-29"

keywords: elasticsearch enterprise, elasticsearch platinum, elasticsearch plans, features

subcollection: databases-for-elasticsearch-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Plan overview
{: #elastic-offerings}

{{site.data.keyword.databases-for}} offers two Elasticsearch services: {{site.data.keyword.databases-for-elasticsearch}} Enterprise and {{site.data.keyword.databases-for-elasticsearch}} Standard. Both plans provide you with a fully managed platform and a scalable Elasticsearch service, allowing you to focus on your applications and data rather than the underlying infrastructure. {{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan deploys the Basic version of Elasticsearch. {{site.data.keyword.databases-for-elasticsearch}} Platinum Plan deploys the Platinum version of Elasticsearch.

**{{site.data.keyword.databases-for-elasticsearch}} Platinum** is only available on Isolated Compute.

{: note}
**{{site.data.keyword.databases-for-elasticsearch}} Feature Availability & Elastic Licensing — Know Before You Enable**

Before enabling any Elasticsearch feature in your environment, review the official Elastic subscriptions page: https://www.elastic.co/subscriptions
Elastic periodically introduces features under Technical Preview or Beta status. During this phase, a feature may be accessible regardless of your current subscription tier — including on lower-tier or free plans. However, this accessibility is not guaranteed to be permanent.
Elastic reserves the right to, without prior notice, graduate a feature from Technical Preview into a paid licensing tier. Customers on lower-tier subscriptions who are actively using that feature at the time of graduation may experience service disruption or loss of access.

| Scenario | Risk |
|---|---|
| Feature is in Technical Preview today | May be moved to a paid tier without notice |
| You use it on a Basic/Free plan | Access may be revoked upon graduation to a higher tier |
| No proactive mitigation in place | Potential production disruption at an unplanned time |

**Recommendations:**

- Verify the licensing tier of any feature before building a dependency on it
- Treat any feature not yet at GA on your subscription tier as a **temporary entitlement**
- Monitor Elastic's release notes and subscriptions page regularly for changes
- Avoid building production dependencies on Technical Preview or Beta features unless your subscription tier will cover them at GA


{: note}



{: note}

Our strategic partnership with [Elastic](https://www.elastic.co/about/){: external} since January 2023 means that we are able to offer more and richer functionality, as well as world-class levels of support.

{{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan does not deploy Elasticsearch Enterprise version.
{: note}

All {{site.data.keyword.databases-for-elasticsearch}} clusters on any plan can make full use of other Elastic Stack components, such as [Kibana](/docs/databases-for-elasticsearch?topic=databases-for-elasticsearch-getting-started#kibana), [Logstash](https://www.elastic.co/logstash/){: external}, and [Beats](https://www.elastic.co/beats/){: external}.

## {{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan
{: #es-enter-plan}

{{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan has all the key functionalities of the Elasticsearch Service, such as [Role Based Access Control (RBAC)](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/es-security-principles.html#security-create-appropriate-users){: external} and [Index Lifecycle Management (ILM)](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/index-lifecycle-management.html){: external}, security, alerting, monitoring, reporting, and graph capabilities.


## {{site.data.keyword.databases-for-elasticsearch}} Platinum Plan
{: #es-platinum-plan}

{{site.data.keyword.databases-for-elasticsearch}} Platinum Plan offers all the features of the Enterprise Plan, plus a richer array of functionality around integrations (connectors and web crawlers), security features (logging and access control), document-level security, and machine learning capabilities (anomaly detection, data frame analysis, and inference and model management), among others.

Access to Platinum features on an IBM deployment is primarily determined by the accessibility of a feature through the Elasticsearch API and the need for modifications to configuration files. Configuration files update is not currently supported for IBM deployments.

For questions regarding specific feature support, submit a [support ticket](https://cloud.ibm.com/unifiedsupport/cases/add){: external}.
{: note}
