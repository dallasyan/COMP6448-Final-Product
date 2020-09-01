---
title: OilRig Analysis
linktitle: OilRig Analysis
type: book
# Prev/next pager order (if docs_section_pager enabled in params.toml)
weight: 3
---

# Oil Rig

The OilRig consortium was an espionage motivated group whose activities were prevalent throughout the period of May 2014-2018. During this time they were relatively successful in achieving their goals. Unit 42, a counter cyber crime intelligence organisation, was tasked with monitoring OilRig’s operations. The OilRig group was classified by such counter threat organisations as a low in attack and operational sophistication but identified as being extremely persistent in the pursuit of their goals. Thus they can still be considered as a significant threat to organisations. 

The group employed evolving attack strategies backed by an internal software development life cycle which worked to increase the functionality of tools and resources used in attacks. In this way the group sought to reduce their detectability and operational predictability. Unit 42 and other groups, as well as the general public, were given a deeper insight into the internal functioning of the OilRig group after a twitter account surfaced which appeared to oppose the OilRig group. The twitter account titled ‘@Mr_L4nnist3r’ posted links to supposed dumps related to internal OilRig affairs. One of the dumps in particular provided a great deal of insight into the groups operations, simply called ‘Glimpse.rar’, contained the following:

+ Tools used by the OilRig group
+ Internal names and versioning for tools
+ Stolen credentials
+ Links to webshells
+ Backdoors
+ Exposed login endpoints

The tools used by the OilRig group, that were revealed by the leak, can be mapped to the names used by SOC && blue teams and consequently to Miter to provide a more in depth analysis of the groups activity. 

![OilRig Group Internal Tool Names Mapped](/assets/OilRig/OilRigTable.png "OilRig Internal Tool Names To Miter")

From the leak versioning could also be observed and tools compared to ones in previous leaks. This analysis highlighted the evolution of OilRigs attack patterns and strategies. Another interesting analysis performed by the unit 42 group was that of the files uploaded to Virustotal. The virustotal site runs uploaded files against a broad variety of antivirus softwares and reports if a particular vendor detects the sample as malware. From the upload history and the leaks unit 42 was able to observe the attackers adapting their attack payloads to be undetectable by antivirus. 

This demonstrates the nature of APTs, even less funded groups, to evolve and adapt their strategies in order to be more effective in their operations and to achieve their goals. Ultimately this analysis also highlights why even smaller non funded groups should not be underestimated by SOCs and blue teams and should be treated as a legitimate threat.

More information on OilRig can be found [here](https://malpedia.caad.fkie.fraunhofer.de/actor/oilrig).

UNIT42 has a more in depth analysis of OilRig [here](https://unit42.paloaltonetworks.com/behind-the-scenes-with-oilrig/).