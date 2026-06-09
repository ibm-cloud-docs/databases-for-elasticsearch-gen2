---
copyright:
  years: 2026
lastupdated: "2026-06-09"

keywords: elasticsearch enterprise, elasticsearch platinum, elasticsearch plans, features

subcollection: databases-for-elasticsearch-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Plan overview
{: #elastic-offerings}

{{site.data.keyword.databases-for}} currently offers one Elasticsearch service: {{site.data.keyword.databases-for-elasticsearch}} Enterprise. This plan provides you with a fully managed platform and a scalable Elasticsearch service, allowing you to focus on your applications and data rather than the underlying infrastructure. {{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan deploys the Enterprise license version of Elasticsearch.

## {{site.data.keyword.databases-for-elasticsearch}} feature availability and Elastic licensing: before you enable
{: #elastic-before-enabling}

Before enabling any Elasticsearch feature in your environment, review the official [Elastic subscriptions page](https://www.elastic.co/subscriptions).

Elastic periodically introduces features under Technical Preview or Beta status. During this phase, a feature might be accessible regardless of your current subscription tier, including on lower-tier or free plans. However, this accessibility is not guaranteed to be permanent.

Elastic reserves the right to, without prior notice, graduate a feature from Technical Preview into a paid licensing tier. Customers on lower-tier subscriptions who are actively using that feature at the time of graduation might experience service disruption or loss of access.

| Scenario | Risk |
|---|---|
| Feature is in Technical Preview today | Might be moved to a paid tier without notice |
| You use it on a Basic or Free plan | Access might be revoked on graduation to a higher tier |
| No proactive mitigation in place | Potential production disruption at an unplanned time |
{: caption="Feature scenarios" caption-side="top"}

### Recommendations
{: #elastic-recommendations}

- Verify the licensing tier of any feature before building a dependency on it.
- Treat any feature not yet at GA on your subscription tier as a **temporary entitlement**.
- Monitor Elastic's release notes and subscriptions page regularly for changes.
- Avoid building production dependencies on Technical Preview or Beta features unless your subscription tier will cover them at GA.

Our strategic partnership with [Elastic](https://www.elastic.co/about/){: external} since January 2023 means that we are able to offer more and richer functionality, as well as world-class levels of support.

All {{site.data.keyword.databases-for-elasticsearch}} clusters on any plan can make full use of other Elastic Stack components, such as [Kibana](/docs/databases-for-elasticsearch?topic=databases-for-elasticsearch-getting-started#kibana), [Logstash](https://www.elastic.co/logstash/){: external}, and [Beats](https://www.elastic.co/beats/){: external}.

## {{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan
{: #es-enter-plan}

The {{site.data.keyword.databases-for-elasticsearch}} Enterprise Plan provides all the key functionalities of the Elasticsearch service available under an Elastic Enterprise license, including the following:

- Security capabilities, including Role Based Access Control (RBAC), field- and document-level security, single sign-on (SAML, OpenID Connect, Kerberos, and JWT), attribute-based access control, Elasticsearch and Kibana audit logging, and encryption at rest support.
- Data management capabilities, including Index Lifecycle Management (ILM), searchable snapshots, snapshot lifecycle management, and data tiers.
- Alerting capabilities, including Watcher and machine learning anomaly detection rule types.
- Monitoring capabilities, including full-stack monitoring across the Elastic Stack.
- Reporting capabilities, including PDF and PNG reports.
- Graph capabilities, including graph exploration and analytics.
