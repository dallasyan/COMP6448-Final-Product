---
title: Running Attacks
linktitle: Running Attacks
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---

This section will detail some possible ways to plan and run attacks on your machine or network.

## Attack Plan

Before running your attacks, it is a good idea to create an attack plan first. [Here](/assets/Red_Team_Techniques.xlsx) is an example attack plan written by our team.\
You should mark down the results of each technique while running your attack, so you can compare what you detect later with the attacks that actually worked to see how much of the attack you were able to detect and prevent.\
While writing your attack plan, it may be useful to group similar techniques together, and also to list their MITRE ATT&CK number next to them. Again, this will make it easier to identify areas of weakness in the analysis stage.

If you aren't sure about how to write your attack plan, it may be useful to look at the next section first.

## Ways to Run Attacks

If you have previous experience in using a variety of tools you can use those to run your attacks. Otherwise, the following are some ways to run attacks. For this exercise it is recommended that you stick to one method to make it easier to search through logs later. Note that you should be running your attacks from a separate machine over the internet to get full logs.

### Invoke-Atomic Red Team

[Invoke-Atomic Red Team](https://github.com/redcanaryco/invoke-atomicredteam) allows you to run attacks directly based on MITRE ATT&CK techniques. If you haven't written your attack plan and plan on using Atomic Red Team, you can look through the [available tests](https://github.com/redcanaryco/atomic-red-team/tree/master/atomics) and base your plan off them. The GitHub Wiki page for Invoke-Atomic Red Team is very comprehensive so I won't write a tutorial on how to use it here. Invoke-Atomic Red Team may be the easiest way to run a test attack, however we require a Linux machine for BPF logs, and it may not be as effective as it would be on a Windows machine.

### Kali Linux

[Kali Linux](https://www.kali.org/) is a Linux distribution designed for digital forensics and penetration testing, and includes a variety of tools for different purposes. Using Kali will require a lot more research and trials than Invoke-Atomic Red Team but is also a lot more customisable. I recommend you look through the Mitre ATT&CK framework, choose some techniques you want to run and pick tools which match those techniques. The [Kali tutorials](https://www.kali.org/category/tutorials/) may be a good place to get started, but you will need to do a lot of searching on your own.

## Documenting Results

You should take notes during your attack. What succeeded and what failed? Why did techniques fail? What could be the impact of the techniques that succeeded?