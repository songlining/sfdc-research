# SFDC Research Learnings

## Contact (2026-02-25)

**Useful fields:** Id, Name, Title, Email, Phone, Department, Job_Role__c, Contact_Lead_Status__c, OwnerId
**Relationships:** Links to Account via AccountId
**Working queries:**
- `SELECT Id, Name, Title, Email, Phone, Department, Job_Role__c, Contact_Lead_Status__c FROM Contact WHERE AccountId = 'xxx' ORDER BY Name` — full contact list for an account
**Gotchas:**
- Duplicate contacts are common (e.g., same person with multiple records and slightly different titles)
- Some "contacts" are mailboxes/team aliases, not individuals
- Job_Role__c is a picklist with values: C-Level / VP, Manager, Individual Contributor, Blank/Unassigned
- Contact_Lead_Status__c values seen: Opportunity, Name, SAL - Working, SAL - Responded, MAL

## Opportunity (2026-02-25)

**Useful fields:** Id, Name, StageName, Amount, ACV__c, New_ACV__c, CloseDate, Type, Primary_Product__c, Product_Families__c, Product_s_Targeted__c, ForecastCategoryName, Probability, OwnerId, Sell_Type__c, Transaction_Type__c, IsClosed
**Relationships:** Links to Account via AccountId
**Working queries:**
- `SELECT Id, Name, StageName, Amount, ACV__c, New_ACV__c, CloseDate, Type, Product_Families__c, ForecastCategoryName, Probability FROM Opportunity WHERE AccountId = 'xxx' AND IsClosed = false ORDER BY CloseDate ASC` — open opportunities for an account
**Gotchas:**
- Type values seen: Renew
- Sell_Type__c values seen: Hashi Paper
- Product_Families__c is a semicolon-separated multipicklist

## Opportunity (2026-03-02)

**Useful fields:** Name, StageName, CloseDate, Type, Primary_Product__c, Product_Families__c, Product_s_Targeted__c
**Working queries:**
- `SELECT Id, Name, StageName, CloseDate, Amount, ACV__c, Type, Primary_Product__c, Product_Families__c, Product_s_Targeted__c FROM Opportunity WHERE AccountId = 'xxx' AND IsClosed = true AND StageName LIKE '%Won%' ORDER BY CloseDate DESC LIMIT 500` — fetch all won deals, then keyword-filter locally for training/RSA/RSE services
**Gotchas:**
- `LIKE` is invalid on multipicklist fields such as `Product_Families__c` (error: `invalid operator on multipicklist field`); use `includes(...)` for exact token matching or fetch then filter in JSON.
