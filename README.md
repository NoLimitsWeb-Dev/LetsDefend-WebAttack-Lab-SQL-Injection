# LetsDefend-WebAttack-Lab

## How to Detect and prevent different types of Web Attacks: SQL Injection,  Cross Site Scripting,  Command Injection,  IDOR,  RFI & LFI and File Upload (Web Shell)

## Practice with SOC Alerts
---
### 🔗115 - SOC165 - Possible SQL Injection Payload Detected
---

## Executive Summary
On **October 7, 2026**, this investigation was launched following a high-severity alert triggered by a corporate Web Application Firewall (WAF) environment. An external threat actor attempted to exploit an input validation flaw via an automated **SQL Injection (SQLi)** attack vector. 

This document serves as an exhaustive, step-by-step forensic walkthrough detailing how the alert was investigated, how open-source threat intelligence (OSINT) was leveraged to profile the adversary, how the raw payload was decoded and evaluated, and the logic used to determine the final system impact.

---

## Step-by-Step Tactical Walkthrough

### Step 1: Alert Triage & Triage Validation
The lifecycle of this incident began in the **Monitoring Channel** dashboard of the SIEM platform. 
* **The Alert:** System flagged `EventID 115` under the rule descriptor **SOC165 - Possible SQL Injection Payload Detected**.
* **Initial Assessment:** The system classified this under the **Web Attack** category with a **High Severity** weighting. Because SQL Injection attempts possess the ability to read, modify, or delete sensitive administrative database tables, the alert was immediately claimed to prevent simultaneous processing by other analysts.
* **Playbook Initiation:** The incident was migrated into the **Case Management Channel**, creating an active incident ticket to systematically lock ownership and establish an investigation audit trail.

---
<img width="1914" height="902" alt="image" src="https://github.com/user-attachments/assets/ac785e81-5bcc-4c95-9f85-afd6120961b5" />

* Click Take Ownership <img width="67" height="48" alt="image" src="https://github.com/user-attachments/assets/a0f5ff95-b7e2-4414-bfeb-0dba09d37241" /> on the Main Channel, to take ownership of the alert case and this will move the alert into the Investigation Channel.


<img width="1916" height="708" alt="image" src="https://github.com/user-attachments/assets/0706a59a-453a-48a5-a0c8-6a51140e8ce3" />

<img width="1537" height="427" alt="image" src="https://github.com/user-attachments/assets/a4d759ca-9964-4397-9e51-62eb2413678d" />

<img width="1908" height="901" alt="image" src="https://github.com/user-attachments/assets/2ffc8ace-b618-4626-83bf-1ee7a4cfac83" />

<img width="1919" height="906" alt="image" src="https://github.com/user-attachments/assets/38ab3bc4-093f-4a97-a1c2-60c9720462dc" />

<img width="1903" height="643" alt="image" src="https://github.com/user-attachments/assets/1a0b9280-095b-4904-b5fb-8ef9e6158d07" />

<img width="1221" height="851" alt="image" src="https://github.com/user-attachments/assets/4043ef39-62e4-4610-bd18-bb9b053051dd" />

<img width="1886" height="736" alt="image" src="https://github.com/user-attachments/assets/1cd2a02e-893c-460c-a247-f93e6b6b45b1" />

<img width="1909" height="870" alt="image" src="https://github.com/user-attachments/assets/f62a63d6-802a-47b6-8d58-c55ca76e4e19" />

<img width="1898" height="882" alt="image" src="https://github.com/user-attachments/assets/82796893-c419-4373-8fbd-9ecbc63580e7" />

<img width="1894" height="832" alt="image" src="https://github.com/user-attachments/assets/3ed200a9-9032-485e-887d-c583512b11ed" />

<img width="1860" height="846" alt="image" src="https://github.com/user-attachments/assets/eacb4d0a-37cd-4629-a931-77e04884ac13" />

<img width="992" height="722" alt="image" src="https://github.com/user-attachments/assets/96c3b31e-4394-4096-8e5d-0c8ae0662438" />


<img width="1912" height="889" alt="image" src="https://github.com/user-attachments/assets/c2665384-ed6e-4bfb-948c-4cb3bccd585d" />

<img width="1905" height="877" alt="image" src="https://github.com/user-attachments/assets/ae202107-4204-400c-b02c-c8930ab95e18" />
<img width="1212" height="897" alt="image" src="https://github.com/user-attachments/assets/724ff868-d4ca-46db-95cb-ba47d5b5943a" />

<img width="1886" height="900" alt="image" src="https://github.com/user-attachments/assets/af27095c-3b9a-45b8-8b5d-eaec1523c504" />
<img width="1211" height="902" alt="image" src="https://github.com/user-attachments/assets/4d52c4c2-6ed5-4030-8916-d0faabf024c2" />

<img width="1880" height="869" alt="image" src="https://github.com/user-attachments/assets/f33b7e8a-42ac-4c81-b6ce-8f3c0ced3e80" />
<img width="1213" height="848" alt="image" src="https://github.com/user-attachments/assets/30afd2d7-b0c9-41ed-8a1e-9f9b0d873a51" />

<img width="1893" height="888" alt="image" src="https://github.com/user-attachments/assets/88b57900-0037-45d7-884a-7a315bcfaca9" />
<img width="1215" height="903" alt="image" src="https://github.com/user-attachments/assets/2c5fd519-be2e-46ac-8c76-9181d60d4b42" />

<img width="1893" height="857" alt="image" src="https://github.com/user-attachments/assets/facf91d8-eed7-4f16-96d4-809a0d801152" />
<img width="1207" height="901" alt="image" src="https://github.com/user-attachments/assets/5b07f3bd-df92-4440-9057-fb4214c6bf69" />

<img width="1893" height="740" alt="image" src="https://github.com/user-attachments/assets/043ffaf1-4ae3-417a-8784-fa3fa23a26e7" />

<img width="1904" height="680" alt="image" src="https://github.com/user-attachments/assets/428bfebc-f8af-49e4-a9bf-b46fb3a0bfe1" />

<img width="1910" height="702" alt="image" src="https://github.com/user-attachments/assets/0737c526-af9a-4471-bdbc-1c08fd1e84f5" />

<img width="1912" height="770" alt="image" src="https://github.com/user-attachments/assets/a28ceba1-e924-434a-9dd6-08377adbc350" />

<img width="1894" height="662" alt="image" src="https://github.com/user-attachments/assets/d29d3a76-1e71-413b-a05f-d2ff472cdc25" />

<img width="1849" height="857" alt="image" src="https://github.com/user-attachments/assets/a6bffdd5-7145-4d0f-9b7d-c140e92ee914" />

---

<img width="1004" height="660" alt="image" src="https://github.com/user-attachments/assets/f655a3a8-0711-45ff-b951-7349aac7ab9f" />

### Artifact Data to Enter

• First Artifact Row:
  • Value: 167.99.169.17
	• Type: Select IP Address from the dropdown menu.
	• Comment: Attacker IP address launching SQL injection payloads.

• Second Artifact Row (Click the '+' icon to create):
	• Value: https://172.16.17.18/search/?q=" OR 1 = 1 -- -
	• Type: URL Address
	• Comment: The last SQL Injection in the web attack.

• Third Artifact Row (Click the '+' icon to create):
	• Value: api.ecreaup.pro
	• Type: Let's select Email-Domain (Since this is the closest network identifier).
	• Comment: Malicious hostname tied to the attacker's infrastructure.

<img width="990" height="669" alt="image" src="https://github.com/user-attachments/assets/d409264a-12d0-4756-93b1-2d65ebf9f645" />


<img width="1001" height="582" alt="image" src="https://github.com/user-attachments/assets/82d2806d-470f-41bb-a962-7937fa34a803" />

```
The alert SOC165 triggered due to a possible SQL Injection payload targeted at Host name: WebServer1001 with IP Address: 172.16.17.18 from the external IP address 167.99.169.17 (api.ecreaup.pro-A Malicious hostname tied to the attacker's infrastructure). 

Upon analyzing the logs, the payload was decoded to: q=" OR 1 = 1 -- -. Threat intelligence queries via VirusTotal and AbuseIPDB confirmed the source IP is highly suspicious, with thousands of abuse reports and malicious classifications, originating from a DigitalOcean hosting pool.

Further analysis of the HTTP transaction history revealed that the web application responded with HTTP 500 Internal Server Error codes for the malicious requests. There is no evidence of an HTTP 200 OK success state, anomalous outbound data volume, or subsequent system compromise. 

Therefore, the incident is classified as a True Positive, Malicious, but Unsuccessful exploit attempt. No host containment is required at this time, but the source IP should be blocked on the perimeter firewall. And no need to perform escalation to Tier 2.
```

<img width="1012" height="388" alt="image" src="https://github.com/user-attachments/assets/4f0560db-657c-43d0-9fad-3938280aa61a" />

<img width="1915" height="888" alt="image" src="https://github.com/user-attachments/assets/9585555b-6c2a-41cd-a6c6-52e2c038171c" />

<img width="1902" height="806" alt="image" src="https://github.com/user-attachments/assets/10bd6ebe-67d0-4344-82fe-366213633ba7" />

<img width="1917" height="609" alt="image" src="https://github.com/user-attachments/assets/109e4d6b-d66e-4a6e-8150-271a3dd57349" />
<img width="1899" height="634" alt="image" src="https://github.com/user-attachments/assets/b119c0e2-19c6-472a-a748-caae036d45ce" />
<img width="1902" height="696" alt="image" src="https://github.com/user-attachments/assets/07c0335d-ea77-4cad-bc04-a2a8ef6827a7" />
<img width="1910" height="316" alt="image" src="https://github.com/user-attachments/assets/0014f703-b76b-4aae-b417-e677cd0d30be" />



