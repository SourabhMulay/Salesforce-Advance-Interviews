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

``` apex
sObjectAccessDecision securityDecision = Security.stripInaccessible(
  AccessType.READABLE,
  [select name, amount, industry from account order by name]
);
```

So it will first check object level permission and then it will look for FLS beaucase if you have access to object and not have access to fields, you cannot access fields.

What if we wanted to have DML on the record!!!

So Schema.sObjectType.Account.isCreatable() will provide you the object level check so before inserting it you'll check if you have create acess to the record of that object.

StripInaccessible works in the same case so if the user have the access to the fields those fields only will get populate!!

```apex

account acc=new account();
acc.name='saurabh';
acc.amount=12;
acc.industry='Pam';


sObjectAccessDecision securityDecision = Security.StripInaccessible(
  accessType.CREATEABLE,

  new lit<account>{acc}
);

insert securityDecision.getrecords()[0];

```

So In this case it will strip off those fields which user do not have access with and save the record with values which they have access with.


## Record Sharing: 

With Sharing:  The class will enforce Record level Security

Without Sharing : No record level security enforced

Inherited Sharing: Inherited from parent, with sharing if entry point!! if class defined with inherited sharing and it's entry point then the access is with sharing.

no sharing clause: Inherited from parent, wihtout sharing if entry point!!! except for lightning! (Components)

LDS: Lightning Data Service adapters enforce the CRUD, FLS and sharing.



