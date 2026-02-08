+++
date = '2026-02-08T09:34:39-06:00'
draft = false
title = 'M365 Search by Author'
+++
## How to search documents by author in Microsoft 365

I found an interesting behaviou detail while using SharePoint KQL to search documents created by a specific person.
Let say you use a search query like "Author=John".

Microsoft says that it "Returns content items authored by John Smith" [Microsoft KQL](https://learn.microsoft.com/en-us/sharepoint/dev/general-development/keyword-query-language-kql-syntax-reference). What does that mean "authored"?. There are two properties at every item - "Created By" and "Modified By" (no "Authored By") and a history of document versions.
So I used to interpret this as "created or updated by John".

It turned out - by "Author=John" search returns only documents created by John or last updated (last modified) by John. 
So the search ignores document updates history and uses only document "final" (or current) version metadata.

