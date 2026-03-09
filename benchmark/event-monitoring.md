### Salesforce Logging Architecture

Salesforce provides multiple categories of monitoring and logging data with different levels of detail and retention. These logging mechanisms fall into three primary categories.

#### 1. Built-in Audit Logs (Available in All Salesforce Orgs)

Every Salesforce organization includes several standard audit data sources that record authentication activity and administrative changes. These logs are automatically generated and cannot be disabled.

Examples include:

- **Login History (`LoginHistory`)** – records login attempts, including source IP address, login status, and authentication method.
- **Setup Audit Trail (`SetupAuditTrail`)** – records administrative configuration changes within the environment.
- **Field History Tracking** – optional feature that records historical field value changes for selected objects.
- **API Usage Metrics** – aggregate counts of API usage available through the Salesforce limits APIs and administrative dashboards.

These logs provide baseline visibility but typically do not contain sufficient detail for advanced security monitoring or forensic investigations.

---

#### 2. Event Monitoring Logs (EventLogFile Dataset)

Salesforce provides enhanced activity logging through **Event Monitoring**, which generates detailed event logs stored in the `EventLogFile` dataset.

A limited set of Event Monitoring logs is available **by default** in Enterprise, Unlimited, and Performance editions with **1-day retention**. These include:

- Login
- Logout
- API Total Usage
- Apex Unexpected Exception
- CORS Violation
- CSP Violation
- Hostname Redirect events

The **Event Monitoring add-on** expands this capability by providing dozens of additional event types and extending retention up to **one year**, depending on licensing.

Examples of additional event types include:

- API request logs
- Report access
- Dashboard usage
- File downloads
- Lightning page navigation
- Apex execution
- URI activity

These logs provide detailed telemetry required for advanced security monitoring, anomaly detection, and forensic investigation.

---

#### 3. Real-Time Event Streams (Streaming Events)

Salesforce also exposes certain monitoring events as **real-time event streams** through the Streaming API and Event Bus.

Examples include:

- API anomaly detection events
- credential stuffing detection events
- file activity events
- Lightning URI activity

These streaming events allow external monitoring systems or SIEM platforms to receive near real-time security telemetry. They are separate from the historical EventLogFile dataset and are primarily used for active threat detection rather than retrospective analysis.



### SBS-MON-001: Enable Event Monitoring Log Storage

**Control Statement:** Organizations using Salesforce Event Monitoring must ensure that storage of required Event Monitoring logs is enabled for all event types necessary to support the organization's security monitoring and compliance policies.

**Description:**  
Salesforce Event Monitoring generates detailed activity logs in the `EventLogFile` dataset. These logs provide visibility into user behavior, API activity, report usage, file access, and other system interactions.

Salesforce automatically generates many event logs; however, organizations must ensure that **storage is enabled for the event types they intend to retain and analyze**.

Event Monitoring log storage determines whether Salesforce retains generated event logs so they can be accessed through the platform or exported to external monitoring systems.

Examples of Event Monitoring log types include:

- API activity logs
- API Total Usage logs
- Report access logs
- Dashboard access logs
- URI navigation logs
- Lightning page view logs
- Apex execution logs
- File download logs

If storage is not enabled for a given event type, Salesforce may not retain the corresponding event data, preventing organizations from analyzing that activity.

Organizations should ensure storage is enabled for all event types necessary to support security monitoring, incident response, and compliance requirements.

**Risk:** <Badge type="warning" text="High" />  
Failure to ensure Event Monitoring log storage is enabled creates a significant monitoring gap.

Without these logs:

- Security teams cannot reconstruct user activity during a suspected breach.
- Data exfiltration events (such as large file downloads or abnormal API usage) may go undetected.
- Detailed forensic investigations become impossible due to lack of historical telemetry.
- Organizations may fail to meet regulatory or internal security logging requirements.

Because Salesforce logs cannot be retroactively generated, failing to retain required event logs results in permanent loss of security telemetry.

**Audit Procedure:**  
1. Navigate to **Setup**
2. Open **Event Monitoring Settings**
3. Verify that **Generate Event Log Files** is enabled
4. Open **Event Manager**
5. Review each event type and verify that **Enable Storage** is enabled for all log types required by the organization's security monitoring policy

**Remediation:**  
1. Navigate to **Setup**
2. Open **Event Monitoring Settings**
3. Enable **Generate Event Log Files**
4. Open **Event Manager**
5. For each required event type:
   - Select the event
   - Enable **Storage**

Organizations should also implement automated export or integration with external monitoring platforms where appropriate.

**Default Value:**  
Salesforce provides a limited set of Event Monitoring logs by default in Enterprise, Unlimited, and Performance editions. These logs include Login, Logout, and API Total Usage events with **1-day retention**.

Additional event types and extended retention (up to one year) require the **Event Monitoring add-on**.

### SBS-MON-002: Retaining Event Logs

**Control Statement:** Organizations must retain security event logs for the defined retention period and implement measures to protect the logs from tampering and unauthorized deletion to ensure forensic availability.

**Description:**  
The retention of security event logs is critical for effective incident investigation and forensic analysis. Security incidents often remain undetected for extended periods of time, and investigators must be able to reconstruct historical activity to determine the origin, scope, and impact of an attack.

Salesforce provides multiple categories of logs with different native retention periods.

Built-in audit logs include:

- **LoginHistory** – records login attempts and authentication activity. Retention is up to approximately 6 months, though high-volume environments may experience shorter effective retention.
- **SetupAuditTrail** – records administrative configuration changes. Retention is 180 days.

These built-in logs provide baseline visibility but typically lack the detail required for advanced security investigations.

Organizations requiring detailed user activity telemetry must use **Event Monitoring**, which provides the `EventLogFile` dataset. These logs include detailed activity such as:

- API activity
- report access
- dashboard usage
- file downloads
- Lightning page navigation
- Apex execution
- URI activity

Retention of Event Monitoring logs depends on licensing:

- **Without the Event Monitoring add-on:** Salesforce provides a limited set of Event Monitoring logs (including Login, Logout, and API Total Usage) with **1-day retention**.
- **With the Event Monitoring add-on:** many additional event types become available and retention is extended, typically ranging from **30 days up to one year**, depending on the license.

Because Salesforce automatically purges logs once their retention period expires, organizations with longer retention requirements must export these logs to an external system.

Common approaches include forwarding logs to:

- a Security Information and Event Management (SIEM) platform
- centralized log aggregation systems
- secure long-term archival storage

External retention ensures that forensic data remains available even after Salesforce purges the logs from the platform.

Salesforce also provides administrative capabilities that allow authorized users to delete stored Event Monitoring data. To reduce the risk of log tampering, organizations must strictly control and monitor permissions related to Event Monitoring data deletion.

Changes to the **"Delete Event Monitoring Data"** permission and related configuration settings should be monitored through the `SetupAuditTrail` log.

**Risk:** <Badge type="warning" text="High" />  
Failure to implement adequate log retention and protection creates significant risks for incident response and regulatory compliance.

If logs are not retained for a sufficient period:

- Security teams may be unable to reconstruct the timeline of an attack.
- Indicators of compromise may be permanently lost before detection occurs.
- The organization may be unable to determine the scope of data exposure.

In addition, attackers who obtain administrative privileges may attempt to delete stored event logs to conceal malicious activity. Without secure external copies stored in a separate system, this could permanently destroy critical forensic evidence.

Because Salesforce logs cannot be retroactively generated, failure to retain logs results in permanent loss of investigative data.

**Audit Procedure:**  
1. Obtain the organization's security policy defining required log retention periods.
2. Review the Salesforce organization's licensing and Event Monitoring configuration to determine the native retention period of logs.
3. Verify whether Event Monitoring logs are exported to an external system such as a SIEM or centralized logging platform.
4. Confirm that the combined retention period (Salesforce native retention plus external storage retention) meets or exceeds the required organizational or regulatory retention policy.
5. Review the `SetupAuditTrail` log to confirm that changes to Event Monitoring configuration and permissions are monitored, including assignment of the **Delete Event Monitoring Data** permission.

**Remediation:**  
1. If longer retention is required, purchase Salesforce Event Monitoring or export logs to an external secure storage system.
2. Implement automated log export or forwarding to a SIEM or centralized monitoring platform.
3. Restrict the **Delete Event Monitoring Data** permission to a minimal set of trusted administrators.
4. Configure monitoring alerts to detect any assignment of this permission or modification of Event Monitoring settings.
5. Periodically verify that log exports are functioning correctly and that required log types are being successfully captured.

**Default Value:**  
Typical native Salesforce retention periods include:

- **LoginHistory:** up to approximately 6 months (may be shorter in high-volume environments)
- **SetupAuditTrail:** 180 days
- **Event Monitoring logs (`EventLogFile` dataset):** a limited set of logs is available by default with **1-day retention**; extended retention and additional log types require the Event Monitoring add-on

### SBS-MON-003: Monitor for Suspicious Logins

**Control Statement:** Organizations must continuously monitor and alert on anomalous login patterns to promptly detect and mitigate compromised accounts and application credentials.

**Description:**  

Continuously monitoring user and application login activity is paramount for maintaining the security of a Salesforce environment, especially for Orgs containing sensitive data. The primary objective is the early detection of compromised credentials, which are often the initial vector for sophisticated data breaches. Effective monitoring requires organizations to move beyond simple success/failure log tracking and implement sophisticated anomaly detection techniques.

Organizations must establish a baseline of normal login behavior for all users and connected applications. Any deviation from this baseline should trigger an alert for investigation. Key anomalous patterns to continuously monitor include:

1. Geographic and Travel Anomalies: Logins originating from an unexpected or uncommon geolocation, particularly from high-risk regions. A high-severity alert should be generated for "impossible travel" scenarios, where a user's successive login attempts originate from two distant locations within a time frame that makes physical travel unfeasible.
2. Suspicious Network Origins: Logins from IP addresses associated with known malicious or anonymity-enabling networks. This includes TOR exit nodes, commercial/consumer VPNs, and proxies, which attackers use to mask their true location and identity.
3. Time-based Anomalies: Logins occurring outside of a user's typical working hours (e.g. in the middle of the night) or during a period when the user is known to be on vacation.
4. High-Volume Failed Logins: A large number of failed login attempts followed by a successful one (a common pattern for brute-force or credential stuffing attacks), particularly if the successful login originates from a previously unseen IP address.
5. Client and User-Agent Changes: Unexplained shifts in the login application (e.g. logging in via a mobile app when only web access is typical) or a suspicious, malformed, or blacklisted user-agent string.

Due to the volume and complexity of login data, this monitoring is typically performed by processing Salesforce `LoginHistory` and relevant Event Monitoring logs in a dedicated monitoring system.


**Risk:** <Badge type="warning" text="High" />  

Failure to continuously monitor and rapidly respond to suspicious login activity creates an immediate and severe risk of a full system compromise. This control's absence exposes the organization to three critical, cascading risks:

1. The primary risk is that a successful suspicious login is the initial foothold for a sophisticated attack. Without automated, anomaly-based alerting, the attacker's presence goes unnoticed, leading to a prolonged "dwell time" (the period between compromise and detection). A longer dwell time allows the attacker to extensively reconnoiter the environment, escalate privileges, and establish persistence. The longer the intruders have to prepare, the harder it is to evict them from the Org.
2. Compromised credentials grant attackers direct access to sensitive customer data, intellectual property, and internal records. The lack of timely detection enables the attacker to perform mass data exfiltration via API, reporting, or export functionality before the account can be disabled, leading to financial, regulatory, and reputational damage.
3. An attacker with elevated access can modify or delete critical configuration settings, business-critical data, or code, leading to system downtime, operational failures, and a loss of data integrity. 


**Audit Procedure:**  
1. Verify the organization utilizes a continuous analytics solution that looks for anomalies in Salesforce login data
2. Confirm that the analytics solution's monitoring scope includes all user logins and all non-human integration or application user logins, regardless of whether the authentication method is Single Sign-On (SSO) or direct Salesforce login.
3. Confirm that there is a documented, mandatory procedure to periodically (e.g. quarterly) review, test, and update the login anomaly detection rules and baselines to ensure they remain effective and minimize false positives. Review should also include an analysis of investigated suspicious login alerts during the review period – no investigations can mean the rules are not firing.
4. Review the process and history of investigating and responding to suspicious login alerts.

**Remediation:**  
1. Take into use a dedicated Security Information and Event Management (SIEM) or a specialized login anomaly detection platform. If a platform is already in place, ensure all Salesforce authentication and event logs, particularly LoginHistory and relevant Event Monitoring logs, are being successfully exported, ingested, and correctly parsed.
2. Configure the monitoring solution to explicitly analyze login events from all sources, including:
- Human Users: Both Single Sign-On (SSO) and direct Salesforce credentials.
- Non-Human/Integration Accounts: Any connected systems, APIs, or automated processes.
3. Periodically review and test the effectiveness of the deployed detection rules, minimizing false positives and updating baselines as organizational travel patterns or integration accounts change.
4. Define clear, high-priority, and time-bound Standard Operating Procedures (SOPs) to investigate and respond to all generated suspicious login alerts.

**Default Value:**  
Salesforce natively provides the raw login data in the LoginHistory object. However, without additional paid features or products or custom in-house development, Salesforce does not automatically detect or alert on the sophisticated, anomalous login patterns described in this control. Detection is entirely manual, relying on an administrator running reports against the raw LoginHistory data.

### SBS-MON-004: Monitor for Suspicious API Activity

**Control Statement:** Organizations must continuously monitor and alert on all API activity to establish a baseline, detect anomalous and malicious activity, and identify potential application and integration abuse in a timely manner.

**Description:**  

Monitoring of API activity is a critical security layer that is necessary even after robust login monitoring is in place. Login monitoring alone is insufficient because many advanced threats — including compromised services, session hijacking, stolen access tokens, and malicious insider activity — can bypass the initial authentication step. To effectively detect these post-authentication attacks, organizations must continuously analyze access patterns against established baselines of normal behavior.

This requires ingesting and analyzing granular API event logs, which are primarily available through the Salesforce Event Monitoring add-on. Key anomalous patterns that must be continuously monitored, baselined, and alerted upon include:
1. Unexpected Data Access/Objects: An application or user suddenly querying, modifying, or deleting objects they have never interacted with before (e.g. a Sales user starting to query security configurations or an integration account exporting massive volumes of a sensitive object like Case or Account).
2. Anomalous Volume of API Calls: Any sharp, unexplained spike or drop in API request volume from a specific user or application, can be an indicator of mass data exfiltration or other malicious activity.
3. Uncharacteristic Action Types: A pattern shift from read-heavy operations to write/delete-heavy operations, such as a reporting application suddenly creating or deleting users, which points to application abuse or compromise.
4. Suspicious Network Origins (Post-Login): API activity originating from an IP address or geolocation that is unusual for the user or application, especially when the access token may have been stolen and is being used from a suspicious location.
5. Use of Unapproved/Deprecated API Versions: Calls being made using API endpoints or versions that the organization has explicitly phased out or restricted for security reasons.
6. Unexpected applications: Application use by users should be monitored for anomalies. It should be noted that the attackers can use common applications and try to hide in plain sight in the logs. They can also use "evil twin" attacks pretending to be a service used by the company but in reality using a different account or even a completely fake tool.

**Risk:** <Badge type="warning" text="High" />  
Failure to monitor API activity creates a critical blind spot for post-authentication attacks, leading to severe and cascading consequences:
1. The primary risk is that a compromised application, integration account, or stolen session token can be used to perform mass, high-speed data exfiltration. 
2. Attackers can leverage API access to perform unauthorized and undetectable manipulation of the Salesforce Org. 

**Audit Procedure:**  

1. Verify the organization utilizes a continuous analytics solution (e.g. SIEM, log aggregation platform) that is verifiably integrated with and ingesting all required API activity data, specifically the detailed Event Monitoring logs from Salesforce.
2. Confirm that the monitoring scope includes all API access, including user-initiated, non-human/integration accounts, and all API methods (e.g. REST, SOAP, Bulk). Verify the logs ingested are granular enough to track specific API calls (CRUD operations) against specific objects.
3. Request evidence (e.g. SIEM rule configurations, simulated alert outputs) to verify that the following specific, high-severity anomaly detection rules are active and tuned:
- Mass Data Exfiltration: Alerts on an anomalous spike in data retrieval volume for a specific user or application, especially for sensitive objects.
- Unauthorized Object Access: Alerts on a user or integration account attempting CRUD operations on objects they have no historical precedent for interacting with (e.g. Sales user accessing Security Settings).
- Suspicious Action Shift: Alerts on a shift from a baseline of read-only operations to a high volume of write or delete operations by a user or application.
- Suspicious Network Origins: Alerts on API calls originating from a suspicious or high-risk geolocation/IP address that is unusual for the accessing entity.
4. Process Review - Triage & Response: Examine the documented procedures for triage, investigation, and response to API anomaly alerts. Review a sample of anomaly findings and the actions taken on them for e.g. the past 6 months, to confirm that alerts are being acted upon within the defined parameter.


**Remediation:**  
1. Deploy or build a dedicated analytics solution for granular API log analysis. Ensure all relevant Salesforce Event Monitoring logs for API activity are being successfully exported, ingested, and correctly parsed into the monitoring platform.
2. Configure the analytics solution to first establish a baseline of normal API behavior for every user and integration account. Following the baseline period, configure and tune high-severity anomaly detection rules to detect deviations, specifically rules that:
- Alert on an anomalous spike in data retrieval (read/query) volume.
- Monitor for CRUD (Create, Read, Update, Delete) operations on objects or security settings outside of an entity's established historical interaction baseline.
- Flag a change from read-heavy activity to an unusual volume of write or delete operations.
- Alert on API calls originating from a suspicious or atypical geolocation/IP address for the accessing entity.
3. Define clear, high-priority, and time-bound Standard Operating Procedures (SOPs) to investigate and respond to all generated API anomaly alerts. The procedure must include immediate steps to lock or deactivate the compromised user or application account and revoke stolen access tokens to prevent further data loss or system manipulation.
4. Establish a mandatory, recurring (e.g. quarterly) review process to:
- Periodically test the effectiveness of the deployed detection rules using simulated incidents (e.g. table-top exercises).
- Review and update the established baselines as new integrations are deployed or organizational usage patterns change.

**Default Value:**  
Salesforce provides some raw API usage data (APITotalUsage), but it does not automatically detect, baseline, or alert on sophisticated API anomalies. Automated anomaly detection requires paid add-ons (like Event Monitoring), third-party solutions, or custom in-house development.

### SBS-MON-005: Monitor API Usage Against Limits

**Control Statement:** Organizations must implement continuous, real-time monitoring and alerting on current API usage against defined Salesforce limits to proactively prevent service disruptions.

**Description:**  
Monitoring API usage against defined organizational and Salesforce-imposed limits is a critical operational security measure to ensure service availability. These limits, which operate on a rolling 24-hour window and represent a total, shared quota across all users and integrated applications in the Org, are designed to ensure system stability and fair resource allocation.

When the organizational API call quota is exceeded, the Salesforce platform begins to block all further inbound API requests until the rolling 24-hour count drops below the threshold. This results in immediate and severe business process disruptions, as core integrations are effectively disabled. Examples of process failures include:

- Inability to sync critical data (e.g. financial or delivery information) between ERP/external systems and Salesforce.
- Interruption of vital business processes (e.g. orders or customer service tickets failing to be created or updated).
- Failure of marketing automation or data warehousing tools to ingest or extract necessary information.

Limit breaches can occur for two primary reasons:
1. Gradual Overconsumption: Poor architectural design or uncontrolled integration growth, where the cumulative API usage of all systems slowly exceeds the Org's capacity.
2. Sudden Spike: A sudden, catastrophic spike in calls caused by a misconfigured internal or third-party application e.g. getting stuck in a loop.

To proactively mitigate these risks, organizations must implement a continuous, real-time monitoring solution. This solution must track current API consumption against the official limits (which are tied to the Salesforce Edition and user license count) and provide immediate alerts before the critical threshold is breached. Limits and current usage can be reviewed in the "System Overview" section under Setup. It is also important to monitor sub-quotas for specific services like Bulk API, Streaming API, and Platform events.

When the API limit is at risk (e.g. above a defined threshold, for example, 80% or 90% utilization), the organization must execute a defined, high-priority incident response plan. This plan must first focus on immediate triage to identify the source of the consumption spike — whether it is a malicious actor, a runaway integration, or unexpected growth. While root cause is still under investigation immediate mitigation steps include:
1. Traffic Reduction: Temporarily reducing API calls by disabling non-critical integrations or their respective system users to conserve remaining quota.
2. Quota Increase: Proactively contacting Salesforce to purchase additional, temporary or permanent API quota if architectural changes are not immediately feasible.

**Risk:** <Badge type="tip" text="Moderate" />  
Failure to stay within API limits creates an immediate and severe risk to the availability of critical business operations. Exceeding the rolling 24-hour API quota blocks all further inbound requests, which effectively disables core integrations (e.g. with ERP), leading to catastrophic failures of vital business processes like order placement and resulting in direct financial loss.

**Audit Procedure:**  
1. Review the organization's daily API limit and current 24-hour usage in the Salesforce System Overview in Setup.
2. Verify that a continuous monitoring solution (at minimum Salesforce's native API Usage Notifications) is implemented and active.
3. Confirm that the monitoring solution is configured to trigger immediate, high-priority alerts at a proactive threshold (e.g. 80% or 90% utilization) before the hard limit is breached. Request evidence (e.g. rule configurations, test alerts) to support this.
4. Examine the formal, documented incident response plan for API limit breaches. Verify the plan clearly defines:
- The immediate triage steps to identify the runaway process or application causing the spike.
- The mandatory, temporary mitigation steps (e.g. disabling non-critical integrations).
- The process for escalating and requesting a temporary or permanent API quota increase from Salesforce.
5. Review the history (e.g. for the past 12 months) of any API limit breach or near-breach incidents. Confirm that the response procedure was followed and was effective in mitigating the service disruption.

**Remediation:**  
1. Configure Salesforce's native "API Usage Notifications" in Setup and, more critically, integrate API consumption data into an external monitoring solution. Configure high-priority alerts to trigger at a proactive utilization threshold (e.g. 80-90%) before the hard limit is reached.
2. Establish a formal, high-priority Standard Operating Procedure (SOP) to be executed upon a high-utilization alert. This SOP must clearly define and be rehearsed to:
- Immediately identify the user, application, or integration responsible for the spike in API consumption.
- Mandate the immediate, temporary disabling of non-critical, high-volume integrations to conserve remaining quota.
- Outline the process for proactively requesting a temporary or permanent API quota increase from Salesforce.
3. Following any API limit breach or near-breach, mandate a post-incident review to analyze the root cause. The outcome should be a plan to re-engineer high-volume integrations for more efficient usage (e.g. using Bulk API or other low-volume methods) to prevent recurrence.
4. Continuous Monitoring Maintenance: Establish a mandatory, recurring (e.g. quarterly) review process to confirm the ongoing integrity and functionality of the API usage data export to the monitoring platform.


**Default Value:**  
Salesforce provides native API limit and usage data, including basic API Usage Notifications. However, it does not automatically implement the proactive, high-priority alerting (e.g. at 80-90% utilization) required to prevent service disruption. Effective control requires manual configuration of native features and/or integration with a monitoring solution.