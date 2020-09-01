---
title: SOFACY Playbook
linktitle: SOFACY Playbook
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---

## Converting ATT&CK Playbooks To Incident Response

In this exercise we used [SOFACY's playbook](https://pan-unit42.github.io/playbook_viewer/?pb=sofacy) described through MITRE ATT&CK techniques to analyse their attacks and how to defend against them.

## What key evolutions happened with this campaign in its 4 main iterations?

### Iteration One

![Iteration One Attack Navigator](/assets/SOFACY/SOFACY_Iteration_1.png "Iteration 1")

### Iteration Two

![Iteration Two Attack Navigator](/assets/SOFACY/SOFACY_Iteration_2.png "Iteration 2")

### Iteration Three

![Iteration Three Attack Navigator](/assets/SOFACY/SOFACY_Iteration_3.png "Iteration 3")

### Iteration Four

![Iteration Four Attack Navigator](/assets/SOFACY/SOFACY_Iteration_4.png "Iteration 4")

### SOFACY Overview

![Iteration Final Attack Navigator](/assets/SOFACY/SOFACY_Full_Cycle.png "Iteration Final")

Over the course of the campaign we see SOFACY make heavy (repetitive) usage of a number of techniques highlighted in green. We see the groups primary evolutions within the Persistence, Privilege Escalation and C2 tactics. 

Overall we see a shift in the way SOFACY conducts Command and Control with Standard Application Layer Protocol and Ingress Tool Transfer (Remote File Copy) being continually suplemented with other C2 techniques over the various iterations. 

In general the group starts using a sophisticated network of tactics and techniques in the inital iteration of their campagin using persistence and privilege escalation techniques as well as an array of Collection methods and Comand and Control techniques. They then move to less sophisticated attacks over the second and third iterations of the groups campagin, which are concentrated at the inital access and post explotation tactics columns of the ATT&CK matrix. In the final iteration of their campagin SOFACY uses a moderately sophisticated attack patern which is still significantly less sophisticated than that of the orginal iteration.

Ultimately this highlights a shift in the groups priorities from launching attacks at broad array of targets (countries) using sophisticated combinations techniques to try to stay undetected, connected and privilaged, on compormised systems while collecting and exfiltrating a significant amount of data from various sources, to less sophisticated attacks concentrated on inital access and post exploitation mainly focused at targets in the US. In the final iteration of the SOFACY group's attack campagin we see a spike in sophistication with a number of techniques and tactics from the first phase of the campagin returning.  

In addition to the evolutions taking place in terms of techniques we also see them changing over a relatively short amount of time (Febuary - November) based off this and the targets (governments, defense, media) SOFACY went after it is likely that the group is well funded, probably state funded, with relatively high cababilities. This is supported by the group making modifications to existing malware and changing malware used throughout the various iterations. We do however also see indicators of lower sophistication, such activity is showcased in the third iteration of the attack where a programinc error in the inital malware delivery scripts meant that the attack failed.

Overall the rapid way in which the group evolved and changed techniques and tactics over the course of the four iterations highlights that the group is likely of high sophistication and relatively difficult as an APT to defend against.                     

## What datasets/data feeds would we need to have coming into the SIEM to detect SOFACY?

Since all the attacks by the SOFACY campagin group originate from Spearphising Attachments (T1566) which typically deliver either Cannon or Zebrocy malware we can classify this as one of SOFACY's high priority techniques. 

Thefore data feeds from:

+ Anti-virus
+ Process command-line parameters
+ Process monitoring
+ Endpoint/network monitoring/sensing
+ Windows logs
+ Detonation chamber
+ Email gateway
+ File monitoring
+ Mail server
+ Network intrusion detection system
+ Packet capture

could help detect SOFACY. Of these however perhaps:

+ Anti-virus
+ Process monitoring
+ Endpoint/network monitoring/sensing

would be among the most important/useful to detect this part of SOFACY's attack pattern, since the other data feeds may become too polluted to be useful given that office document based attatchments are primarily being used which may be hard to test given the volume they appear in within organisations.

However detection of SOFACY through data feeds related to Spearphishing attatchments alone may not be enough since they are a sophisticated APT. Thus secondary feeds for SOFACY's other priority techniques like Ingress Tool Transfer (T1105) are likely necessary to detect SOFACY. 

Data feeds from:

+ File monitoring
+ Netflow/Enclave netflow
+ Network protocol analysis
+ Process use of network  

are likely to be necessary to detect this C2 and Lateral Movement behaviour (as classified by the UNIT 42 playbook).

## Would we use static correlation or user/entity behavior analytics, or both?

The usage of both static correlation and user/entity behaviour analytics is by far the most affective approach. This is because of the way UEBA strengthens static analysis by reducing alert fatigue and false positives while providing greater context to security incidents and creating a broader analysis net by inclduing non-technical aspects of organisations like human behaviour. 

### SIEM Rules For Static Analysis

To write a potentially effective SIEM rule would require correlation of several diffrent logs from accross security architecture in the organisation. To write an affective SIEM rule for detecting a SOFACY intrustion/attack we may aim to write a SIEM rule that correlates logs from the mail server/gateway with anti-virus logs and windows logs to detect a SOFACY office document based payload executing in the network.  

![table](/assets/SOFACY/table.png "SIEM Rule Table")

[Possible SIEM rule]

### Defining User/Entity Behaviours && Populations

First and foremost for establishing User/Entity Behaviours a strong and accurate baseline behaviour should be defined for the various user/entity populations. This is perhaps the most important behaviour to define since it determines how well the UEBA system will work.

Baseline behaviours may include (but are certainly not limited to):


+ Where users normally login to specific systems
+ What time a user normally logs into a system
+ What time a user normally connects to the internal network
+ What systems the user typically interacts with within the internal network
+ What time a user normally logs out of a system
+ What time a user normally leaves the internal network
+ What kind of device(s) a user typically connects to the internal network with     

User/entity populations of interest may include:

+ Privileged users like system administrators, top-level administrators
+ Non-technical users (users who don't interact significantly with many systems) like HR
+ Remote users, users who typically connect to systems outside of the internal network  

Populations are important because the influence how significantly a deviation of their normal behaviour should be treated. That is for a privileged user a small deviation from normal behaviour is generally more likely and therefore a minor devation will not be ranked as highly by the UEBA system vs a non-privileged user deviating from normal behaviour, which would be considered more significant. 

### Interaction of the models

At its focus UEBA is designed to work most powerfully as a way of strengthening static analysis. Therefore the models should interact in a way where static analysis is cross refrenced with UEBA in doing so alert fatigue can be reduced and the accuracy of potential threats/security incidents can be increased. UEBA should be connected to as many data feeds as possible since the power of UEBA lies in its ability to consider large cross sections of organisations.

## Triaging a Potential SOFACY Intrusion

+ Is the actor still active in the network
	+ System and network discovery comands from windows log
	+ Recent macro enabled document(s) parsing through mail gateway
	+ Unusual behaviour from lower privileged acounts
		+ Remote file copy
		+ Process use of network logs and Network protocol analysis
		+ Network signatures triggered
+ Has the actor been able to exfiltrate any data
	+ Newly registered domain -> windows logs
	+ Base 64 encoded URL strings parsing through firewall



## How could we work ahead of the adversary? 

Given that the SOFACY group makes heavy use of Spearphishing attatchment and User Execution to gain an inital foothold we implement technical controls around these techniques to work ahead of SOFACY. Furthermore given that the group makes heavy use of two malware varients 'Cannon' and 'Zebrocy', in the later iteration of their campagin, we can configure malware security profiles around these.

+ Antivirus/Antimalware software with signature rules in place 

This is likely to work relatively well as a technical control against SOFACY however if the payloads do not match known signatures the AV will need to trigger on unusual macro behaviour. 

However, putting technical controls in place that only aim to stop the attack group gaining inital access is not a good strategy. Thus technical controls that limit techniques used heavily internally by SOFACY are also important to work ahead of the adversary. Such a technical control could be defining firewall rules that block outbound traffic with base 64 URL encoding. 

Since a priority technique used by SOFACY for both lateral movement and C2 is 'Remote File Copy' or in ATT&CK 'Ingress Tool Transfer' 'Network Intrusion Prevention' mitigations could be configured with signatures specifically designed to detect this activity.

In general knowing where the attacker is 'up to' in their attack cycle/pattern is also exeedingly useful to working ahead of the adversary, provided we have data related to the group from prior attacks/activity from another organisation or our own. Since with this data we can predict the kind of techniques that we should expect to see used next and thus tailor/configure security systems and rules to halt the adversary's progress.   
