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

---
<img width="1914" height="902" alt="image" src="https://github.com/user-attachments/assets/ac785e81-5bcc-4c95-9f85-afd6120961b5" />

---
* Click Take Ownership <img width="67" height="48" alt="image" src="https://github.com/user-attachments/assets/a0f5ff95-b7e2-4414-bfeb-0dba09d37241" /> on the Main Channel, to take ownership of the alert case and this will move the alert into the Investigation Channel.
* In the Investigation Channel -- Action -- Click >> to Create the Ticket

<img width="1537" height="427" alt="image" src="https://github.com/user-attachments/assets/a4d759ca-9964-4397-9e51-62eb2413678d" />
<img width="1916" height="708" alt="image" src="https://github.com/user-attachments/assets/0706a59a-453a-48a5-a0c8-6a51140e8ce3" />

* Details of the alert
---
<img width="1908" height="901" alt="image" src="https://github.com/user-attachments/assets/2ffc8ace-b618-4626-83bf-1ee7a4cfac83" />

* Click on the **Continue Button**

---
<img width="1919" height="906" alt="image" src="https://github.com/user-attachments/assets/38ab3bc4-093f-4a97-a1c2-60c9720462dc" />

* Click on the **OK Button**

---
<img width="1903" height="643" alt="image" src="https://github.com/user-attachments/assets/1a0b9280-095b-4904-b5fb-8ef9e6158d07" />

* **Playbook Initiation:** The incident was migrated into the **Case Management Channel**, creating an active incident ticket to systematically lock ownership and establish an investigation audit trail.

---
### Step 2: Traffic Path Analysis & Asset Scoping
---

<img width="1886" height="736" alt="image" src="https://github.com/user-attachments/assets/1cd2a02e-893c-460c-a247-f93e6b6b45b1" />

---
### Mapping out the environmental scope of the connection using network connection details:
<img width="1916" height="708" alt="image" src="https://github.com/user-attachments/assets/0706a59a-453a-48a5-a0c8-6a51140e8ce3" />

<img width="1894" height="832" alt="image" src="https://github.com/user-attachments/assets/3ed200a9-9032-485e-887d-c583512b11ed" />

1. **Directionality:** Traffic originated externally from the Internet and targeted an internal corporate demilitarized zone (DMZ) segment.
2. **Attacker Host:** `167.99.169.17` (Source)
3. **Internal Target:** `172.16.17.18` (Destination)
4. **Target Asset Profile:** Cross-referencing the destination IP within the **Endpoint Security** center identified the host as `WebServer1001`. The machine runs a 64-bit architecture built on **Windows Server 2019** and is managed via the `webadmin` administrative profile.

<img width="1909" height="870" alt="image" src="https://github.com/user-attachments/assets/f62a63d6-802a-47b6-8d58-c55ca76e4e19" />

5. **Port Audit:** The connection hit port **443 (HTTPS)**, verifying that the attack payload was wrapped inside encrypted SSL/TLS layers to bypass basic signature-matching perimeter sensors.
<img width="1912" height="889" alt="image" src="https://github.com/user-attachments/assets/c2665384-ed6e-4bfb-948c-4cb3bccd585d" />

---

## Step 3: Adversary Profiling via Open-Source Intelligence (OSINT)
To gauge the sophistication of the attacker, external threat infrastructure lookups were executed across multiple global threat intelligence platforms.

### 1. VirusTotal Verification
Querying `167.99.169.17` returned flags from security vendors including BitDefender and G-Data, categorizing the node as a source of active **Phishing / Suspicious** operations. The infrastructure map showed the IP mapping directly to an Autonomous System network pool (cloud hosting pool) controlled by **DigitalOcean, LLC (AS14061)**. 

<img width="1898" height="882" alt="image" src="https://github.com/user-attachments/assets/82796893-c419-4373-8fbd-9ecbc63580e7" />

---

### 2. AbuseIPDB Verification
Abuse trackers revealed an aggressive historical fingerprint. The IP was actively flagged with **14,765 individual consumer and corporate reports** documenting historical automated web application exploitation, brute forcing, and port scanning. The IP resolved to a dynamic hosting address `api.ecreaup.pro` deployed out of a cloud node location in Santa Clara, California.
<img width="1860" height="846" alt="image" src="https://github.com/user-attachments/assets/eacb4d0a-37cd-4629-a931-77e04884ac13" />

<img width="1771" height="797" alt="image" src="https://github.com/user-attachments/assets/b247a67d-1807-42bc-97bd-9c7f4b218900" />
<img width="1894" height="904" alt="image" src="https://github.com/user-attachments/assets/61431f38-8307-4e58-a1e1-b3b6ac80003f" />

#### Strategic Deduction:
The host is not a standard end-user desktop node. It is a cloud-hosted virtual private server (VPS) node repurposed by the threat actor to act as an automated, programmatic scanning proxy. The velocity of past reports points to the use of automated vulnerability search utilities (such as `sqlmap` or custom scanning scripts).

---

<img width="992" height="722" alt="image" src="https://github.com/user-attachments/assets/96c3b31e-4394-4096-8e5d-0c8ae0662438" />

---

### Step 4: Payload Extraction & Cryptographic Decoding
The network logs revealed that the attacker targeted a search utility endpoint. The string was heavily encoded using percent-encoding (URL Encoding) to ensure the malformed characters safely traveled over HTTP without breaking the browser protocol.
<img width="1912" height="889" alt="image" src="https://github.com/user-attachments/assets/c2665384-ed6e-4bfb-948c-4cb3bccd585d" />

* Click **Log Management** on the right side of the LetsDefend
* Type the Malicious IP ```167.99.169.17``` into the search box and also the date ```2022-02-25 ``` to filter

#### Raw Logs:
* Click <img width="55" height="52" alt="image" src="https://github.com/user-attachments/assets/fa0ae7e6-c890-4fb4-90f4-0906170a5fe1" /> **RAW** to output the Raw Log for that Log
* Select and copy the **Request URL**
* Paste the copied **Request URL** into an open source **Cyber Chef** and Decode 

```http
GET /search/?q=%22%20OR%201%20%3D%201%20--%20- HTTP/1.1
Host: 172.16.17.18
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```
<img width="1893" height="857" alt="image" src="https://github.com/user-attachments/assets/facf91d8-eed7-4f16-96d4-809a0d801152" />

#### CyberChef / Decoded Analysis:
Passing the parameter value `%22%20OR%201%20%3D%201%20--%20-` through a standard URL Decoder yields the plaintext injection structure:
```sql
q=" OR 1 = 1 -- -
```
<img width="1207" height="901" alt="image" src="https://github.com/user-attachments/assets/5b07f3bd-df92-4440-9057-fb4214c6bf69" />

#### Deep Dive - Deconstructing the Exploitation Intent:
* **`"` (Double Quote):** The attacker inputs a literal quote character intending to prematurely break out of the application's developer-defined string variable context inside the backend SQL query code block.
* **`OR 1 = 1` (The Tautology):** Because `1 = 1` is a mathematical absolute that evaluates to `TRUE`, appending an `OR` condition forces the entire underlying database conditional logic statement (`WHERE` clause) to evaluate to true, completely bypassing standard query filtering.
* **`-- -` (The Comment Token):** The double-dash instructs the SQL database engine interpreter to ignore all remaining lines of code in the legitimate query string. This effectively deletes any trailing syntax syntax controls or trailing syntax validation checks implemented by the developer.

If successful, this payload would force the application database to return all records within the targeted table rather than filtering for a specific query string. This establishes clear **Malicious** intent, confirming a **True Positive** alert state.

---
<img width="1893" height="740" alt="image" src="https://github.com/user-attachments/assets/043ffaf1-4ae3-417a-8784-fa3fa23a26e7" />

The alert SOC165 triggered due to a possible SQL Injection payload targeted at WebServer1001 (172.16.17.18) from the external IP address 167.99.169.17. Upon analyzing the logs, the payload was decoded to: q=" OR 1 = 1 -- -. Threat intelligence queries via VirusTotal and AbuseIPDB confirmed the source IP is highly suspicious, with thousands of abuse reports and malicious classifications, originating from a DigitalOcean hosting pool.

Further analysis of the HTTP transaction history revealed that the web application responded with HTTP 500 Internal Server Error codes for the malicious requests. There is no evidence of an HTTP 200 OK success state, anomalous outbound data volume, or subsequent system compromise. Therefore, the incident is classified as a True Positive, Malicious, but Unsuccessful exploit attempt. No host containment is required at this time, but the source IP should be blocked on the perimeter firewall.
<img width="606" height="336" alt="image" src="https://github.com/user-attachments/assets/45c6385e-ba82-478e-aa1c-c4412f06763d" />

**Note:** On the raw log, the server returned an HTTP Response Status: 500 (Internal Server Error) after the attacker attempted a SQL Injection vulnerability scan or exploit.

---

## Step 5: Exploit Success & Impact Determination
The defining metric of any web application incident investigation relies heavily on validating host-level success vs. failure. The perimeter logs confirmed the **Device Action** was logged as **Allowed**, meaning the WAF didn't block the connection at layer 7. Therefore, the application logs were audited to verify the internal reaction.

### The Success Metric Checklist:
1. **HTTP Status Code Verification:** The application log records show the targeted web server uniformly answered the payload bursts with an **HTTP 500 Internal Server Error** status code. 
2. **Code Execution Audit:** An HTTP 500 state explicitly validates that the malformed SQL command broke the compilation flow of the database driver script. Instead of evaluating the injected logic and executing it, the web server crashed gracefully at the page level.
3. **Data Exfiltration Audit:** Had the attack successfully bypassed authentication or queried rows, the server would have returned an **HTTP 200 OK** response combined with a disproportionately massive outbound data size packet (high byte count) representing data extraction. The log telemetry showed minimal byte transmission matching standard text error pages.

**Conclusion:** The exploit attempt **Failed / Was Unsuccessful**. The system successfully resisted the input payload due to native application crash limits rather than perimeter active blocking.

---

Based on the raw log analysis from the previous step, the correct choice is **SQL Injection**.

The attack vector used is a single quotation mark ``(%27)``, which is the definitive indicator of a SQL Injection probe used to break database query structures.

<img width="1904" height="680" alt="image" src="https://github.com/user-attachments/assets/428bfebc-f8af-49e4-a9bf-b46fb3a0bfe1" />

---

<img width="1910" height="702" alt="image" src="https://github.com/user-attachments/assets/0737c526-af9a-4471-bdbc-1c08fd1e84f5" />

• No matching emails exist within the Email Security / Mailbox tab indicating a scheduled authorization or penetration test for this activity.

• The source IP address and host do not correspond to any authorized attack simulation products.

* Therefore, Malicious traffic was not caused by a planned test.

<img width="1912" height="770" alt="image" src="https://github.com/user-attachments/assets/a28ceba1-e924-434a-9dd6-08377adbc350" />
<img width="1849" height="857" alt="image" src="https://github.com/user-attachments/assets/a6bffdd5-7145-4d0f-9b7d-c140e92ee914" />

Independently of the SQL Injection probe, the attacker has not gained command execution access or a reverse shell on this specific terminal workspace. The threat remains isolated to the HTTP application layer vulnerability.

---
<img width="1894" height="662" alt="image" src="https://github.com/user-attachments/assets/d29d3a76-1e71-413b-a05f-d2ff472cdc25" />

---

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
<img width="1004" height="660" alt="image" src="https://github.com/user-attachments/assets/f655a3a8-0711-45ff-b951-7349aac7ab9f" />

---
## Step 6: Post-Incident Remediation & Playbook Closing
Because the server was never compromised, emergency asset containment (network layer isolation) was bypassed to preserve the operational uptime of `WebServer1001`. The response vector transitioned to containment of the external source infrastructure.
<img width="990" height="669" alt="image" src="https://github.com/user-attachments/assets/d409264a-12d0-4756-93b1-2d65ebf9f645" />

* No Escalation needed to be perform to Tier 2

---
<img width="1001" height="582" alt="image" src="https://github.com/user-attachments/assets/82d2806d-470f-41bb-a962-7937fa34a803" />

**Analyst Note:**
```
The alert SOC165 triggered due to a possible SQL Injection payload targeted at Host name: WebServer1001 with IP Address: 172.16.17.18 from the external IP address 167.99.169.17 (api.ecreaup.pro-A Malicious hostname tied to the attacker's infrastructure). 

Upon analyzing the logs, the payload was decoded to: q=" OR 1 = 1 -- -. Threat intelligence queries via VirusTotal and AbuseIPDB confirmed the source IP is highly suspicious, with thousands of abuse reports and malicious classifications, originating from a DigitalOcean hosting pool.

Further analysis of the HTTP transaction history revealed that the web application responded with HTTP 500 Internal Server Error codes for the malicious requests. There is no evidence of an HTTP 200 OK success state, anomalous outbound data volume, or subsequent system compromise. 

Therefore, the incident is classified as a True Positive, Malicious, but Unsuccessful exploit attempt. No host containment is required at this time, but the source IP should be blocked on the perimeter firewall. And no need to perform escalation to Tier 2.
```

### Final Incident Disposition:
The ticket was officially submitted to the system queue with a final closed verdict of **True Positive**, classified under malicious intent with an exploitation outcome of **Unsuccessful**. 

---
<img width="1012" height="388" alt="image" src="https://github.com/user-attachments/assets/4f0560db-657c-43d0-9fad-3938280aa61a" />

<img width="1915" height="888" alt="image" src="https://github.com/user-attachments/assets/9585555b-6c2a-41cd-a6c6-52e2c038171c" />

<img width="1917" height="609" alt="image" src="https://github.com/user-attachments/assets/109e4d6b-d66e-4a6e-8150-271a3dd57349" />
<img width="1902" height="806" alt="image" src="https://github.com/user-attachments/assets/10bd6ebe-67d0-4344-82fe-366213633ba7" />


<img width="1899" height="634" alt="image" src="https://github.com/user-attachments/assets/b119c0e2-19c6-472a-a748-caae036d45ce" />
<img width="1902" height="696" alt="image" src="https://github.com/user-attachments/assets/07c0335d-ea77-4cad-bc04-a2a8ef6827a7" />
<img width="1910" height="316" alt="image" src="https://github.com/user-attachments/assets/0014f703-b76b-4aae-b417-e677cd0d30be" />



