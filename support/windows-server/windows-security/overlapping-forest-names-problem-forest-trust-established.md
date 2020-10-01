---
title: Overlapping Forest Names cause problems once Forest Trusts are established
description: Describe how to solve the problem of overlapping Forest Names causes when Forest Trusts are established
ms.date: 09/10/2020
author: Deland-Han
ms.author: delhan 
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Domain and forest trusts
ms.technology: WindowsSecurity
---
# Overlapping Forest Names cause problems once Forest Trusts are established

This article provides the resolution when overlapping Forest Names cause problems once Forest Trusts are established.

_Original product version:_ &nbsp; Windows Server 2012 R2  
_Original KB number:_ &nbsp; 2744558

## Symptoms

You have multiple forests and you have trusts between these forests. You have two forests that have overlapping DNS names like:
forest1.com
forest2.forest1.com
Now when you have a third forest forest3.com that has forest trusts with the two other forests, you can't work with accounts from forest2 in this forest. For example, when you want to add accounts from forest2 to a DACL in forest3, you will encounter this error:
>C:\temp>Icacls c:\temp\test1 /grant forest2.forest1.com\admins:r  
forest2.forest1.com\admins: No mapping between account names and security IDs was done.  
Successfully processed 0 files; Failed processing 1 files

When you print the trusts information regarding suffix routing, you see that the suffix is reported as conflicting:
C:\>netdom trust forest3.com /namesuffixes:forest1.com
   Name, Type, Status, Notes
1. *.forest1.com, Name Suffix, Enabled

C:\>netdom trust forest3.com /namesuffixes:forest2.forest1.com
   Name, Type, Status, Notes
1. *.forest2.forest1.com, Name Suffix, Conflicting, With forest1.com

In a network trace, you can see a Kerberos Ticket request from a user in forest2.forest1.com for a resource in forest3.com fails against a DC in forest3:
231 lsass.exe (708) \<client> \<dc forest3> KerberosV5 KerberosV5:TGS Request Realm: forest3.com Sname: cifs/fileserver.forest3.com233 Idle (0) \<dc forest3> \<client> KerberosV5 KerberosV5:KRB_ERROR  - KDC_ERR_POLICY (12)
In a KDC ETL you will see something like:
DEB_ERROR,dll,pac_cxx3792,KdcFilterSids(),"Failed to filter SIDS (LsaIFilterSids): 0xc000019b".

## Cause

Kerberos requires exact suffix mapping. LSA uses one set of functions for routing domain searches and the Kerberos rules are used there for forest trusts.

## Resolution

When you replace the forest trust between forest3.com and forest2.forest1.com with an external trust, the problem does not happen as there is only an exact mapping of domain names, and no suffix mapping as required by Kerberos.
Another approach avoiding this error is to exclude the suffix of forest2.forest1.com in the forest trust between forest3.com and forest1.com. As the suffix for the "child" forest is in conflict, you need to re-activate this suffix on the "child forest" trust.

## More information

References:
Forest trusts and exclusions: [https://blogs.technet.com/b/askds/archive/2009/04/10/name-suffix-routing.aspx](https://blogs.technet.com/b/askds/archive/2009/04/10/name-suffix-routing.aspx) 
UPN Routing with exclusions: [https://technet.microsoft.com/library/dd560680(v=WS.10).aspx](https://technet.microsoft.com/library/dd560680%28v=WS.10%29.aspx)