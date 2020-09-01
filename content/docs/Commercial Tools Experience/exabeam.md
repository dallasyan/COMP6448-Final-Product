---
title: Exabeam
linktitle: Exabeam
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---

We used [Exabeam](https://www.exabeam.com/) while analysing a red team attack to see what users may have been affected. I believe we used [Exabeam Advanced Analytics](https://www.exabeam.com/product/exabeam-advanced-analytics/). Watching the video gives a good idea of what it's like. Note that we were able to look at data but could not modify anything, so we couldn't make comments on users, or mark them as suspicious.

## Identifying Attacks

The home page had some lists with titles such as "Account Lockouts" or "Notable Users", with users and, depending on the title, a score associated with them. From there you can navigate to the profile of suspicious users. Once on a profile, you can view a timeline of their activity. This made it easy to look for signs of an attack. For example if you know there was an attack run at a specific time, you can look through the timeline of users and check for things such as a high number of login attempts.

Exabeam also adds tags to activities it finds suspicious, noting what kind of attack it could be. For example, if there were a high number of failed login attempts, it might be marked as "brute force".

## Communication Between Analysts

You can add notes to users, for example explaining suspicious activity, or noting that their issue has been resolved. Then if other analysts look at this user they know what activities have already been analysed.

You can also mark users as suspicious. I'm not sure of the process as we couldn't do this ourselves, but doing this means that other analysts know who to keep an eye on for suspicious activity.

## Overall Experience

Overall Exabeam was very easy to use and presented important information in an easily digestible manner. I was also able to look at comments from other analysts to learn more about the activities of users without looking through their timeline. I can definitely see how this would make analysis of attacks a lot quicker and allows analysts to easily work together.

Note that we used Exabeam for a very specific use case (analysis of a single attack), so there are probably a lot of features we didn't use, particularly regarding long term security. Please look at their website or do your own research to learn more.