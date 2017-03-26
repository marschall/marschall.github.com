---
layout: post
title: Oracle Compatible Row Limiting in H2
---
If you are concerned with H2 and Oracle compatibility in your SQL, e.g. because you're using H2 for testing and Oracle for production, you'll be glad to know that H2 [1.4.177](https://www.h2database.com/html/changelog.html) and later support [Oracle row limiting syntax](https://docs.oracle.com/database/122/SQLRF/SELECT.htm#GUID-CFA006CA-6FF1-4972-821E-6996142A51C6__BABBADDD) (12c and later)

```sql
OFFSET x ROWS
FETCH FIRST n ROWS ONLY 
```

Unfortunately this is currently not [documented](https://www.h2database.com/html/grammar.html#select) but this is hopefully fixed in the future.

Know that this is used for pagination it as correctness issues if the result set is updated between two queries and has performance issues for large offsets. For more information check out Markus Winand's article on [keyset pagination](https://use-the-index-luke.com/no-offset) on the subject.

