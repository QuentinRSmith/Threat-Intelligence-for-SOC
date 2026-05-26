# Threat-Intelligence-for-SOC
I will be going through a TryHackMe room to demonstrate how to utilise Threat Intelligence to improve the Security Operations pipeline.

# Introduction

In this room, we’ll focus on how Threat Intelligence strengthens the Security Operations pipeline and how shared information across teams can be used effectively. We’ll also cover who produces and consumes Threat Intelligence, the different types of intelligence, and how it can be applied to prevent and detect malicious activity.

# Threat Intellegence Feed

Threat Intelligence Producers collect, analyse, and share threat data. They gather information through methods like network monitoring, honeypots, and internal incident analysis, then publish reports, advisories, and IOC feeds for others to use. Being a Producer requires large data sets, strong analytical capability, and the resources to identify emerging threats.

Threat Intelligence Consumers use intelligence created by Producers to strengthen their security posture. They rely on shared data to identify vulnerabilities, block or detect malicious activity using IOCs, improve incident response, and collaborate by validating and sharing useful findings.

Whether an organisation is a Producer or Consumer depends on its resources, expertise, and the role its security team plays in gathering versus applying threat intelligence.


First, we will hunt and consume IOCs provided by Threat Intelligence Producers.

We start by starting our VPM and logging in

<img width="776" height="494" alt="image" src="https://github.com/user-attachments/assets/43787d8b-0a73-446a-acca-785ba4ed796a" />


We will now be using uncoder.io which is an online tool that transforms Sigma rules, IOC lists, and other platform query syntaxes into custom hunting queries prepared for execution in SIEM and XDR. It is an easy-to-use tool that could assist us in hunting the following IOCs up for investigation. For IOCs, the tool accepts six different types of IOCs, namely:

IPs
Domains
URLs
Hashes
Emails
Files


<img width="489" height="83" alt="image" src="https://github.com/user-attachments/assets/d5c85bc2-cfbd-480e-ad81-16be54174747" />


Next, we use the following IP list and upload it to Uncoder. After pasting the IOCs, Uncoder will automatically clean defanged IPs and remove duplicates, leaving four unique addresses. 

IP list:
135[.]181[.]103[.]89
185[.]224[.]126[.]215
185[.]224[.]128[.]215
171[.]24[.]136[.]15
171[.]22[.]136[.]15
195[.]133[.]40[.]108
103[.]190[.]37[.]169
103[.]170[.]37[.]169
103[.]190[.]37[.]169
185[.]224[.]128[.]215
107[.]175[.]202[.]151
107[.]175[.]202[.]158
195[.]133[.]40[.]108
107[.]175[.]202[.]158
109[.]206[.]240[.]194



<img width="941" height="359" alt="image" src="https://github.com/user-attachments/assets/fa47359a-f02c-4661-be25-57d6d33594ad" />



We come out with the following result:

<img width="402" height="112" alt="image" src="https://github.com/user-attachments/assets/2d6ff633-29d3-4620-957b-94c8544079d7" />



The result of using the tool can be utilised in our Kibana Instance via the Discover feature.


We will paste the destination and add the time frames of 2/14/23 - 2/17/23 in Elastic and come out with the following:

<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/12f74805-dcc5-4817-aa99-5f3a3e1565c9" />

From this, we can see things such as how many connections were made to certain addresses:

<img width="247" height="200" alt="image" src="https://github.com/user-attachments/assets/54c2661a-f355-49f4-8d0b-b262cdb603f5" />

We can see the IP address of the compromised host:

<img width="211" height="203" alt="image" src="https://github.com/user-attachments/assets/03586016-746d-4168-b20f-df4e3a00d7ec" />

Lastly, we can see the destination port of IP: 107.175.202.151:

<img width="548" height="424" alt="image" src="https://github.com/user-attachments/assets/4025a009-fb4d-4961-ad0c-531c04abe60f" />


# Intelligence-Driven Prevention

Now in this room:
Your organisation has identified itself as a consumer of Threat Intelligence, meaning your role is to use intelligence from trusted sources to deploy security controls and prevent threats in your environment. Using the IOCs provided by these sources, you can strengthen your defences by blocking or detecting malicious activity.

To begin, it helps to simplify the common types of IOCs found in threat feeds:

Domains – Often linked to malicious hosting, C2 servers, or spam activity.

IP Addresses – Typically associated with known attack sources or malware callbacks.

These IOCs form the basis of the controls you will apply to protect your infrastructure.

**IP Block Via Firewall**
IP blocking is a common security control that stops inbound or outbound traffic based on the IP address attempting the connection, typically enforced through firewall rules. While firewall configuration can feel complex, blocking known malicious IPs is an effective first step. Doing so helps prevent intrusive connections that could disrupt services or exploit vulnerabilities, and it also stops infected systems from reaching a threat actor’s infrastructure after malware execution.

Next we will use SIEM logs to hunt for any activity involving domains flagged as malicious by your DNS Sinkhole. Starting with the provided domain agrosaoxe[.]info and run searches using the KQL templates below by replacing the placeholders with the domain and your sinkhole IP:

dns.question.name: "agrosaoxe.info"

dns.answers.data: "sinkhole_IP_here"


We see that there are 11 DNS queries to agrosaoxe[.]info that has been created

<img width="944" height="501" alt="image" src="https://github.com/user-attachments/assets/ee66d40e-0148-4e5f-a5c9-f81cf9348b26" />

Next, we see the IPv4 addresses are resolved by agrosaoxe[.]info and the IP address used for DNS Sinkhole

<img width="202" height="262" alt="image" src="https://github.com/user-attachments/assets/93798049-35dc-4672-9d7e-300f3cf1f957" />

Lastly, we see how many hits were caused by connections to sinkholed domains which is 115

<img width="949" height="394" alt="image" src="https://github.com/user-attachments/assets/8fd5b837-c63d-40a9-9fae-1c702ae1b9f7" />


# Intellegence-Driven Detection

Our next task in this room: 

You have successfully deployed preventive mechanisms to mitigate known IOCs in your infrastructure. To maximise the capabilities of your detection and response, you are now tasked to improve the detection capabilities of your tooling.

We have started utilising Threat Intelligence from the previous task to prevent potential compromises from malicious actors. Now, we will leverage Threat Intelligence IOCs to know if something suspicious is happening in our infrastructure effectively. 

In this task, we will use the following Sigma rule to hunt for sinkholed domains

The Sigma rule searches for DNS queries that resolve to 0.0.0.0, which often indicates a domain blocked by your DNS Sinkhole. Seeing this resolution in logs can signal attempts to reach a known malicious domain based on your sinkhole configuration.

Use Uncoder.io to convert the Sigma rule into an ElastAlert format. Set the conversion direction to Sigma → ElastAlert, then click Translate to generate the ElastAlert version of the rule.

<img width="935" height="470" alt="image" src="https://github.com/user-attachments/assets/a1d6bbad-abc7-4385-950a-5f74e571d748" />

To begin working with ElastAlert, I connected to the machine via SSH using the provided credentials and navigated to the ~/elastalert directory. This directory contains the main config.yaml file, which defines how ElastAlert connects to the Elasticsearch instance, and a rules subdirectory that includes a placeholder rule named sinkhole.yaml.


<img width="739" height="458" alt="image" src="https://github.com/user-attachments/assets/d026ff18-3821-4137-890e-d7a9154eff3d" />



I replaced the contents of sinkhole.yaml with the ElastAlert rule generated from Uncoder.io after converting the Sigma rule. Before saving it, I made two required adjustments:

I removed everything in the description line starting from Author: to ensure valid syntax.

I updated the index value from winlogbeat-* to filebeat-* so the rule queries the correct data source.

After these changes, the rule defined an alert that triggers whenever DNS queries resolve to 0.0.0.0, which indicates a sinkholed domain.

Once the rule was configured, I returned to the main ElastAlert directory and executed ElastAlert using a command that starts evaluation from February 16, 2023, prints verbose output, and writes the results to output.txt. I allowed the process to run until ElastAlert reported that it had completed its initial query cycle, indicated by a summary showing the number of hits, matches, and alerts sent. After that, I stopped the process with CTRL+C.



<img width="692" height="419" alt="image" src="https://github.com/user-attachments/assets/553f721c-a6e5-4b11-a2c3-48980648ae06" />


This completed the setup, configuration, and execution of the ElastAlert rule for detecting sinkholed DNS activity.


# Conclusion

In the previous tasks, we covered the key differences between Threat Intelligence Producers and Consumers and why understanding your organisation’s needs and capabilities is essential for getting value from intelligence. We also explored how to apply intelligence‑driven prevention and detection, using tools like Sigma, Uncoder, and ElastAlert to build practical detection logic.

Overall, this room focused on how Threat Intelligence can strengthen a Security Operations Center by improving both detection capabilities and preventive controls against known threats. In a landscape where threats constantly evolve, organisations must make full use of shared intelligence—whether produced by dedicated research teams or refined through feedback from consumers. Through collaboration and shared knowledge, we can collectively improve the security posture of every organisation.

<img width="550" height="278" alt="image" src="https://github.com/user-attachments/assets/78c033e8-ba23-4189-8d10-97bc0dc50301" />








