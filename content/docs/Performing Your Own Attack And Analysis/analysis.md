---
title: Log Analysis
linktitle: Log Analysis
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

After performing the attacks, you will analyse your logs and see what information you can find, and what is missing. If you have been following this exercise exactly then you will only be able to analyse network packets, and not internal information. Hopefully it will still be able to give you a taste of what it's like though.

## Mapping Techniques

Look through the logs with whatever method you want, and try to identify any MITRE ATT&CK techniques that were used. This may be a bit difficult to do if you have created the attack plan and run your own attacks, because you will know which techniques you used. If possible, get a friend to look through the logs to see what they can find. A good start might be looking at what ports the attack tried to access, and searching up the common use cases of those ports to see what kind of technique may have been used.

Document the overall experience. Was there information you wish you had logged that would've made things easier? Was there so much information you couldn't look through them effectively? If you created alerts, did they catch any useful information? Were your logs okay but you didn't have an effective way to search them?

## Reflection

Compare the techniques you identified during analysis to your attack plan. Which techniques didn't you detect? Did these techniques succeed, and if so what could you have done to prevent and/or detect them? A good start would be looking up the techniques that weren't prevented and/or detected on MITRE ATT&CK and reading the Mitigations and Detection sections.