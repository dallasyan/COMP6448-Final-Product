---
title: LogRhythm
linktitle: LogRhythm
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

We used [LogRhythm](https://logrhythm.com/) to search logs for information on a red team attack on systems. I believe we used [LogRhythm Cloud](https://logrhythm.com/products/logrhythm-cloud/). Note that our access may not have been set correctly. For example, we didn't see any alarms but the analysts said they did. This may have affected our experience.

## Alarms

As we wrote above, we did not receive any alarms so we cannot comment on its usage.

## Searching

Searching for logs was very smooth. Granted, we had a very simple use case, we just wanted to get all logs within a certain time frame. Looking through the searching feature however, it seemed very detailed and if you know what kind of information you're looking for it would be very easy to find related logs. Once you have completed a search, there is also searching within that set of logs. Clicking the search bar on top of a column shows the top results for that column too, so if you are looking for suspicious activity in general it can be useful to see things such as commonly accessed files. We used this feature to look at things such as ports used and objects, which we then looked up on the MITRE ATT&CK Framework to see if they were related to a common attack there.

## Finding Information

Once you have a log set, you can do things like pinning columns so they're always visible, and there are different tabs to look at different kinds of logs. For example, logs from application access. Clicking on a log gives more detailed information on it.

We found it difficult to find information that specifically correlated to attacks until we received the actual attack plan. I don't think this has anything to do with LogRhythm itself, but that we were misusing it by trying to manually find information blind. I think the way it's intended to be used is alarms go off or you find information from other higher level services such as Exabeam. This gives you a basic idea of the attack, and you can then search the logs for more detailed information on them. Unfortunately the alarms didn't seem to work for us which made it difficult to identify attacks. Once we received the attack plan we could search for logs related to each technique listed.

We did come across a problem where if we had a large time frame (24 hours), the logs we received seemed to be an overview as when we made the time frame smaller we received more detailed logs. This is probably not a problem in the majority of cases because you should have an idea of the time frame before looking through logs.

## Overall Experience

Overall, the searching experience was very smooth, but we had trouble actually identifying attacks until we were given more detailed information on it. This is likely due to a combination of factors unrelated to LogRhythm itself, that our alarms weren't working and we were beginners in the field. Once we were able to narrow down our searches, we found a lot more useful information.

Like with the other products, it is likely we only utilised a very small subset of the features of LogRhythm because we had a very specific use case. If you are interested please visit their website or do your own research.