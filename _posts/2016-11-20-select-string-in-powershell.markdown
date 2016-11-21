---
layout: post
title:  "Working with Select-String Matches in Powershell"
author: "David Lewis"
date:   2016-11-20 16:52:00 -0800
---

Grep is one of the most useful Linux commands...

but it doesn't exist natively on a Windows environment. You can always install a Grep utility or via [Chocolatey for Windows](https://chocolatey.org/), However, Powershell has a very useful utility called [`Select-String`][select-string-docs], and it is rather powerful. The most challenging part can be extracting the group from a pattern match, however it is surprisingly simple.

Extract text from a file, set of files, or an object.

{% highlight powershell %}
Select-String [-Pattern] <String[]> [-Path] <String[]> [-SimpleMatch] [-CaseSensitive] [-Quiet] [-List]
 [-Include <String[]>] [-Exclude <String[]>] [-NotMatch] [-AllMatches] [-Encoding <String>]
  [-Context <Int32[]>] [<CommonParameters>]
{% endhighlight %}

{% highlight powershell %}
PS C:\> $STR = Select-String -AllMatches -Pattern 'id: "([a-zA-Z0-9\.]*)"' -Path .\gruntfile.js
{% endhighlight %}
This will give us a Matches object, which when called can give us a list of members and objects:

{% highlight powershell %}
PS C:\> $STR.Matches
Groups   : {id: "OpenApi.Promotion.V1", OpenApi.Promotion.V1}
Success  : True
Captures : {id: "OpenApi.Promotion.V1"}
Index    : 12
Length   : 26
Value    : id: "OpenApi.Promotion.V1"

Groups   : {id: "Nsb.Promotion", Nsb.Promotion}
Success  : True
Captures : {id: "Nsb.Promotion"}
Index    : 12
Length   : 19
Value    : id: "Nsb.Promotion"
{% endhighlight %}
Any of these members you can call directly. (`Groups`, `Success`, `Captures`, `Index`, `Length`, and `Value`)

{% highlight powershell %}
PS C:\> $STR.matches.value
id: "OpenApi.Promotion.V1"
id: "Nsb.Promotion"

PS C:\> $STR.Matches[0].Value
id: "OpenApi.Promotion.V1"

PS C:\> $STR.Matches.Length
2

PS C:\> $STR.Matches.Groups.Length
4
{% endhighlight %}
You might have thought Length would give us something like `26, 29`, but because it's called on the object and there were 2 capture groups found, it gave us a value of 2, and when called on groups a value of 4. That's because each group contained 2 capture groupings, which gives us a hint that calling `$STR.Matches.Groups[1].Value` might not give us what we would initially expect.

{% highlight powershell %}
PS C:\> $STR.matches.groups[1].Value
OpenApi.Promotion.V1

PS C:\> $STR.matches.groups[3].Value
Nsb.Promotion

PS C:\> $STR.matches.groups
Groups   : {id: "OpenApi.Promotion.V1", OpenApi.Promotion.V1}
Success  : True
Captures : {id: "OpenApi.Promotion.V1"}
Index    : 12
Length   : 26
Value    : id: "OpenApi.Promotion.V1"

Success  : True
Captures : {OpenApi.Promotion.V1}
Index    : 17
Length   : 20
Value    : OpenApi.Promotion.V1

Groups   : {id: "Nsb.Promotion", Nsb.Promotion}
Success  : True
Captures : {id: "Nsb.Promotion"}
Index    : 12
Length   : 19
Value    : id: "Nsb.Promotion"

Success  : True
Captures : {Nsb.Promotion}
Index    : 17
Length   : 13
Value    : Nsb.Promotion
{% endhighlight %}

Instead, use the index on each Matches to find the capture group we're looking for:

{% highlight powershell %}
PS C:\> $STR.Matches[0].Groups[1]
Success  : True
Captures : {OpenApi.Promotion.V1}
Index    : 17
Length   : 20
Value    : OpenApi.Promotion.V1

PS C:\> $STR.Matches[0].Groups[1].Value
OpenApi.Promotion.V1
{% endhighlight %}
But what if we want to be able to get or manipulate all of the captured groups? Use [`Foreach-Object`][foreach-object-docs]:

{% highlight powershell %}
ForEach-Object [-InputObject <PSObject>] [-Begin <ScriptBlock>] [-Process] <ScriptBlock[]> [-End <ScriptBlock>]
 [-RemainingScripts <ScriptBlock[]>] [-WhatIf] [-Confirm] [<CommonParameters>]
{% endhighlight %}

{% highlight powershell %}
PS C:\> $STR.Matches | ForEach-Object { $_.Groups[1].Value }
OpenApi.Promotion.V1
Nsb.Promotion
{% endhighlight %}

Text manipulation can be done as part of the object. For example, we'll convert to uppercase and do a string replace to change the periods to underscores:
We'll use the second group since that contains the capture element within each matches object to find the value we want to manipulate.
{% highlight powershell %}
PS C:\> $STR.matches | ForEach-Object { $_.Groups[1].Value.ToUpper() }
OPENAPI.PROMOTION.V1
NSB.PROMOTION

PS C:\> $STR.matches | ForEach-Object { $_.Groups[1].Value.ToUpper() -replace "\.", "_" }
OPENAPI_PROMOTION_V1
NSB_PROMOTION
{% endhighlight %}

[foreach-object-docs]:  https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.core/ForEach-Object
[select-string-docs]:   https://technet.microsoft.com/en-us/library/hh849903.aspx
