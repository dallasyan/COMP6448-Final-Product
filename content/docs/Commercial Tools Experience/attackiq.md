---
title: AttackIQ
linktitle: AttackIQ
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

We used [AttackIQ](https://attackiq.com/platform/) to run a test attack on some servers. We split into two teams, and when one team was attacking the other was looking through the logs for the attack. Note that we were not given direct access to AttackIQ and watched analysts use it while we worked together to decide on which techniques to use and how to configure them for our attack.

## Running Attacks

AttackIQ made it incredibly easy to map our [attack plan](/assets/Red_Team_Techniques.xlsx) to an actual attack, as it used the MITRE ATT&CK Framework technique numbers. All we had to do was choose the techniques we wanted to run, configure them slightly if necessary, then start the attack. We could split the attack into tests, which we used to separate the phases of the attack (discovery, initial access, execution, credential access, privilege escalation, persistence, lateral movement, collection, exfiltration).

## Creating Reports

After the attack is run, you can see feedback on the attack from the website directly, or you can download reports as PDFs. In a manual attack, you would have to take notes during the entire attack which could make things very slow, and if you miss some details that would make analysis difficult later.

### Summary Report

Overall Status shows the total number of techniques used, and how many were prevented and/or detected.

Test Overview shows the per test numbers. It also shows how many times each test was run.

Historical Run Results is a graph of percentage of techniques detected and/or prevented over time.

Threat Assessment is a pie chart showing the percentage of techniques (per test) blocked, not blocked, and other (e.g. errored out).

### Detailed Report

Top Mitigation Recommendations is exactly as it sounds. It provides possible ways to prevent techniques that weren't blocked. It is a table with the mitigation recommendation along with the unblocked techniques it is related to and how many times they occurred.

Scenarios Overview lists the techniques to watch out for, along with their success rate.

Assets Overview lists the success rate of attacks on individual assets, for example if you're running tests on multiple machines in a system.

Scenario Details contains an in depth description of each technique that was used, what the result was for each asset, and mitigation recommendations for the technique. This is the bulk of the report as it is very detailed.

The appendix contains descriptions of the different phases of attacks and information on how to apply each mitigation technique.

## Overall Experience

Overall, the experience with AttackIQ was excellent. It made running the tests incredibly easy as there was nothing that had to be installed onto the machine to run the tests from. The creation of reports was very useful in analysis, and also made running the test a lot faster as we did not have to manually write notes.

Note that we did not spend much time using AttackIQ so there are probably a lot of features that I missed. Please look at their website or do your own research for more information.