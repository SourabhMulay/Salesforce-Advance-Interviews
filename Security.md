# Security

<hr>

**Note:** In every Interview you'll get fucked by these questions tho they are just complicated and no one wanted to remember but still the person infront of you will judge you on the basis of this logic less questions! So ww'll be **learning!**

<hr>

### User Mode Vs System Mode:

The standard UI's the API calls (Rest, SOAP, UI API) then execute anonymous and the standard controllers are RUN in USER MODE! (Means CRUD , FLS and Sharing is Enforced).

The Apex Classes, Apex Triggers, Apex Web services are Run in SYSTEM MODE. (Means CRUD, FLS and Sharing Not Enforced).


### Object and FLS:

It's provided using profiles and permission sets! Enforcing CRUD and FLS.

1. Schema Methods  (SOQL : Read Data) (Modify Data: DML).
2. WITH SECURITY_ENFORCED (it will provide or check SOQL (READ). No Modify data or dml).
3. Security.stripInaccessible() (Both read and DML).
4. Database Operations in USER MODE (pilot) (Both read and DML).


Before querying the Object you can check if it's accessible using schema.sObjectType.Account.isAccessible() or if you wanted to check the fields use Schema.sObjectType.Account.name.isaccessible() and then only query the object.

With Security Enforced those checks are very simple!! So if i select Account or query accounts with security Enforced, if i do not have anu permission over any field i will receive the error. "Query includes inaccessible fields".

StripInaccessible() : So In case user do not have access to some of the fields which they have not have access so it will return null for those.

```
sObjectAccessDecision securityDecision = Security.stripInaccessible(
AccessType.READABLE,
[select name, amount, industry from account order by name]
);
```

