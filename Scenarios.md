# Scenario Based Questions : (AI answered)

## Describe what happens if you import the same Apex method twice in a component , once with @wire and once imperatively?

When you use the same Apex method reference in both a @wire decorator and an imperative call, the behavior depends heavily on the Lightning Data Service (LDS) cache 🗄️.

In a Lightning Web Component, you typically import the method once:
import myMethod from '@salesforce/apex/Controller.myMethod';

Here is what happens when you use that single import in both patterns:

1. The Cache Connection 🔄
If the Apex method is marked as cacheable=true, both the @wire service and the imperative call share the same client-side cache.

The @wire behavior: It manages the data reactively. When the component loads, it automatically checks the cache or calls the server.

The Imperative behavior: When you trigger the method imperatively (like on a button click), LWC first checks if the data is already in the cache from that @wire call. If it is, you get a "cache hit," and the data returns almost instantly without another trip to the Salesforce server.

2. Independent Execution (Non-Cacheable) ⚡
If the method is not cacheable, you actually can't use it with @wire at all! @wire requires the cacheable=true annotation. Therefore, if you are using both, the method must be cacheable, meaning they will always be linked by that shared cache.

3. Identity and Imports 🆔
Technically, if you were to write two identical import statements in your JavaScript file, the engine treats them as the same reference. There is no "double-importing" penalty at the module level; it's all about how you invoke that imported function.

## What happens if a child component's @api property receives an object reference from the parent and the child mutates it directly?

If a child component tries to directly mutate an object received via an @api property from its parent, the LWC framework will throw a read-only proxy error in the browser console, and the change will not take effect.

This happens because LWC enforces a strict "data down, events up" one-way data flow. When the parent passes the object down, the framework wraps it in a read-only proxy to prevent the child from altering the parent's state without the parent knowing.

Let's explore how to handle this correctly. Which of these approaches would you like to look at next?

The LWC Standard: Dispatching a Custom Event so the parent can safely perform the update.

The Local Copy: Creating a clone of the object so the child can manipulate it independently for its own UI.

Let's break down both of these solutions, as they represent fundamental patterns in Lightning Web Components.

1. The LWC Standard: Custom Events 📤
LWC enforces a strict "Data down, Events up" architecture. Instead of the child changing the data, the child asks the parent to change it.

The Child: When the user makes a change, the child component creates a CustomEvent (e.g., updateuser). It packages the new value inside the event's detail property and dispatches it.

The Parent: The parent component listens for this event. When it catches the event, it extracts the new value from the detail property and updates its own data.

The Result: Because the parent's data changed, the LWC framework automatically pushes the fresh data back down to the child.

2. The Local Copy: Cloning 🪞
Sometimes, a child component needs to manipulate data for its own UI purposes, and the parent doesn't need to know about those changes.

The Child: When the object arrives from the parent (usually handled in a setter or connectedCallback), the child creates a separate, unlinked copy of it.

For a simple object, you can use the spread syntax: this.localData = {...apiData}.

For a complex, nested object, you might do a deep clone: this.localData = JSON.parse(JSON.stringify(apiData)).

The Result: The child component mutates this.localData freely without hitting the read-only proxy error, because it is no longer touching the parent's original reference.


## What happens if you call an imperative Apex method inside connectedCallback() without handling the promise rejection?

Let's look at how JavaScript handles Promises in this lifecycle hook.

When you call an imperative Apex method, it returns a Promise. If that Apex method fails (perhaps due to a query limit or a null pointer exception) and you haven't handled the rejection, a couple of things happen:

The Silent UI Failure: The Lightning Web Component framework will not automatically show an error message to the user on the screen.

The Console Error 🛑: The error is swallowed by the UI, but if you open the browser's developer tools, you will see an "Uncaught (in promise)" error in the console.

The Stuck State ⏳: This is the biggest risk. Often, developers set a property like isLoading = true before calling Apex, planning to set it to false when the data returns. If the promise is rejected and unhandled, the code to hide the spinner never runs, and the user is left staring at a perpetually loading screen.

To prevent this, we need to explicitly tell the component what to do if the server returns an error.

If we are writing standard Promise syntax (using .then() for the success response), what block do we need to chain onto the end of it to catch that rejection?

## Describe what happens when two triggers on the same object both perform DML on a related object , what risks does this introduce?

Gemini said
Having multiple triggers on the same object is generally considered an anti-pattern in Salesforce. When two triggers on the same object perform DML on the same related record, it introduces a few critical system risks.

Here are the two biggest dangers ⚠️:

Unpredictable Execution Order 🎲: Salesforce does not guarantee the order in which multiple triggers on the same object will execute.

Row Locking (UNABLE_TO_LOCK_ROW) 🔒: When a record is being updated, the database locks it to prevent other processes from changing it at the exact same millisecond. If both triggers try to update the same related record during a complex or bulk transaction, they can collide, causing the entire transaction to fail with a locking error.

## What happens if a Queueable Apex job is enqueued inside a trigger that fires on a bulk load of 10,000 records?

When you insert or update a large number of records, Salesforce doesn't send them all through the trigger at the exact same time. Instead, to manage memory and performance, it breaks those 10,000 records down into smaller, more manageable chunks.

Do you recall how many records Salesforce processes in a single trigger chunk by default? It's 200!! 

Exactly! 200 records per chunk.

So, if a bulk load of 10,000 records comes in, Salesforce divides it into exactly 50 chunks (10,000 / 200). This means your trigger will execute 50 separate times within that single transaction.

Here is where the risk comes in: If your trigger contains code that blindly enqueues a Queueable job every time it runs, it will try to add 50 jobs to the queue. The Salesforce governor limit for adding Queueable jobs in a single synchronous transaction is exactly 50.

If a user uploads 10,201 records, the trigger fires a 51st time, hits the limit, and the entire transaction crashes with a LimitException 💥.

To fix this, we need a way to gather all the records across those multiple chunks and only process them once. Instead of enqueuing the job immediately inside the trigger, what kind of Apex variable could we use in a helper class to temporarily collect all those unique Record IDs as the chunks process?

Gemini said
Exactly. A static Set<Id> is the right tool because static variables maintain their state across all the individual trigger chunks within a single transaction.

So, we create a static Set<Id> pendingIds and add the new records to it as each chunk runs. To avoid hitting that limit of 50 queueable jobs, we can use a static Boolean isJobEnqueued flag to ensure we only call System.enqueueJob(new MyQueueable(pendingIds)) one single time.

But here is the architectural trap 🪤: When you instantiate a Queueable class and pass data into its constructor, Salesforce immediately takes a serialized "snapshot" of those variables right at that exact moment.

If our trigger enqueues the job during the very first chunk of the bulk load, how many of the 10,000 record IDs will actually end up in the Queueable job to be processed?

No problem! It is a very tricky scenario that catches a lot of developers off guard.

Only the first 200 records would make it into the Queueable job.

Here is why: When you call System.enqueueJob() during that first chunk, Salesforce takes a snapshot of your static Set<Id> right at that exact moment. At that time, it only contains the 200 IDs from chunk #1. As the remaining 49 chunks process, they successfully add their 9,800 IDs to the static Set, but the Queueable job has already been packaged up and sent to the queue. The train has already left the station! 🚂

Because of this, trying to process massive data loads by enqueuing jobs directly inside triggers is usually an anti-pattern. A much safer architecture is to flag those records (e.g., setting a Needs_Processing__c checkbox to true) and having a Scheduled Batch job run in the background to sweep them all up safely.

## What happens if you use String.format() with mismatched placeholders and arguments in Apex?

Scenario 1: Too Many Placeholders 🛑
If your string expects two variables, like 'Hello {0} and {1}', but you only pass a list with one item ['Alice'], the code fails at runtime. Apex tries to find the item at index 1 to replace the {1} placeholder, realizes the list is too short, and throws a fatal System.StringException: Invalid argument at index 1.

Scenario 2: Too Many Arguments 🤔
Now, let's think about the exact opposite situation.

Suppose your template string only has one placeholder: 'Welcome, {0}!'
But, you accidentally pass it a list containing three names: ['Alice', 'Bob', 'Charlie'].

What do you think happens in this case? Does Apex throw an error because of the extra unused arguments, or does it handle them differently?

- No Apex will not throw exception: Apex only throws a System.StringException if it is missing an argument for a placeholder.


## How do you handle pagination when an external REST API returns large datasets across multiple pages in Apex?

Handling large, paginated API responses in Apex is all about navigating Salesforce's strict Governor Limits—specifically, the maximum number of callouts per transaction (typically 100) and the Heap Size limit 📦 (6MB synchronous, 12MB asynchronous).

If an external system has 500 pages of data, a single standard Apex transaction will time out or crash before it finishes fetching and processing them all. To solve this, we have to break the work apart.

The approach we choose depends entirely on the business requirement.

Let's look at a specific backend scenario: Imagine you need to build a nightly process that fetches 200 pages of newly created invoices from an external ERP system and saves them as Salesforce records ☁️.

Because this is a heavy data-processing task that runs in the background, we need a way to make a callout for Page 1, process the data, and then somehow start a brand new transaction to fetch Page 2 using the nextPageToken.

Which asynchronous Apex feature allows you to chain jobs together, where the finish or execute method of one job can safely enqueue the exact same class again for the next page?

Queueable Apex is the perfect tool for this specific scenario.

Here is exactly how that "chaining" works:

Your Queueable class (which implements Database.AllowsCallouts) makes the HTTP GET request for Page 1.

It parses the JSON response and processes the records.

It checks the response for a nextPageToken.

If the token exists, the code takes that token, passes it into the constructor of a new instance of that exact same Queueable class, and uses System.enqueueJob() to put it in the queue.

It is basically a relay race where the baton is the next page token! 🏃‍♂️ (Batch Apex is also great, but usually only if you already know the exact number of pages upfront using a custom Iterable).

## What happens if a Named Credential's authentication token expires mid-transaction during an Apex callout?

If a Named Credential's token expires exactly mid-transaction, the external server will reject the request and return an HTTP 401 (Unauthorized) status code back to your Apex code.

However, how the system recovers depends entirely on how you configured the Named Credential:

With OAuth & Refresh Tokens 🔄: This is where Named Credentials shine. If configured with an Auth. Provider that supports refresh tokens, Salesforce manages the lifecycle automatically. If the platform detects the access token is expired, it will seamlessly use the refresh token to get a new one. But, if the token expires at the exact millisecond the request is in flight, your current Apex transaction will still receive that 401 error.

Without Refresh Tokens (or Basic Auth): The callout simply fails. An administrator or user will have to manually re-authenticate the Named Credential.

To build a truly resilient enterprise integration, your Apex code should check if (response.getStatusCode() == 401) and contain logic to gracefully retry the callout or log the failure for a background job to pick up.

## What happens if two simultaneous inbound API requests try to upsert the same record in Salesforce at the same time?

This creates a classic database race condition! 🏎️

When two API requests try to upsert the exact same record at the exact same millisecond, the outcome depends on the current state of the database:

1. The Row Lock Error 🔒
If the record already exists in Salesforce, the first request will temporarily lock the record to update it. If the second request tries to touch that exact same record before the first one finishes, the second request will fail and return a dreaded UNABLE_TO_LOCK_ROW error.

2. The Duplicate Creation Trap 👯
This is the sneakier scenario. Let's say the record doesn't exist yet, and both requests are relying on an External ID to perform the upsert.

Both requests search the database simultaneously.

Both see that the External ID does not exist.

Both decide to perform an insert.
Because the database index hasn't had time to update from the first request, Salesforce might actually let both inserts succeed, leaving you with two duplicate records instead of one!

To prevent this kind of chaos, enterprise architects often design integrations to use idempotency.

Have you worked with idempotency keys before, or would you like to explore how we use them to ensure a request is only processed once?

Let's break down idempotency.

In computer science, an operation is idempotent if performing it multiple times produces the exact same result as performing it just once.

Here is a simple analogy:

Non-idempotent: Pushing a "Next Track" button ⏭️ on a music player. If you push it three times, you skip three songs. The system state changes with every push.

Idempotent: Pushing a "Pause" button ⏸️. If you push it once, the music stops. If you panic and mash the button three more times, the music is still just paused. The end result is identical.

In the context of APIs and our Stripe payment scenario, idempotency protects us from network hiccups.

When Salesforce initiates a payment, it generates a unique ID (an Idempotency Key 🔑) and sends it in the HTTP header. Stripe receives the request, charges the card, and saves the receipt linked to that specific key.

If the network connection drops before Salesforce gets the receipt, Salesforce will automatically retry the request. Because the retry contains the exact same Idempotency Key, Stripe realizes it already processed this transaction. Instead of charging the card a second time, it simply replies with the saved receipt from the first attempt.

## Write an optimized query to find top 3 performing Accounts per Region based on total Opportunity revenue, using only one aggregation query ?

The trick here is that standard SOQL does not support window functions (like PARTITION BY in standard SQL). This means you cannot apply a LIMIT 3 per region directly inside the database query itself.

To solve this while adhering to the constraint of "only one aggregation query", we have to split the work:

The Database (SOQL): Write a single Aggregate SOQL query that calculates the total revenue for every account, grouped by region, and sorts the results perfectly.

The Server (Apex): Use a loop in Apex to process the sorted results and pluck out just the top 3 for each region.

````
Map<String, List<Id>> topAccountsPerRegion = new Map<String, List<Id>>();

// 1. The Single Aggregation Query
// We sort by Region first, then by the highest Revenue descending.
List<AggregateResult> results = [
    SELECT Account.Region__c region, AccountId, SUM(Amount) totalRevenue
    FROM Opportunity
    WHERE Account.Region__c != null 
    GROUP BY Account.Region__c, AccountId
    ORDER BY Account.Region__c ASC, SUM(Amount) DESC
];

// 2. The Apex Loop 
for (AggregateResult ar : results) {
    String region = (String)ar.get('region');
    Id accId = (Id)ar.get('AccountId');

    // If this is the first time we see this region, create a new list for it
    if (!topAccountsPerRegion.containsKey(region)) {
        topAccountsPerRegion.put(region, new List<Id>());
    }

    // Since the query is already sorted by highest revenue, 
    // the first 3 accounts we process for any region ARE the top 3.
    if (topAccountsPerRegion.get(region).size() < 3) {
        topAccountsPerRegion.get(region).add(accId);
    }
}

System.debug(topAccountsPerRegion);
````


## Your query on OpportunityLineItem is running through 8 million records , how would you improve its performance using custom indexes?

Let's dive into Large Data Volumes (LDV) and SOQL performance ⚡.

When you query against 8 million OpportunityLineItem records, you run the risk of a "Full Table Scan." This means the database engine has to check every single row to see if it matches your WHERE clause, which will quickly time out or hit Salesforce governor limits.

To fix this, we use indexes. Think of an index like the index at the back of a textbook 📚. Instead of reading every page to find a specific topic, you check the index, which tells you exactly which pages to flip to.

Salesforce automatically indexes standard fields like Id, Name, and CreatedDate. For your own fields, you can ask Salesforce Support to create a Custom Index to speed things up.

However, there is a catch! Just having a custom index doesn't mean Salesforce will actually use it. The query optimizer evaluates your WHERE clause to decide if using the index is worth the effort. This concept is called Selectivity.

To make the query optimizer actually use your custom index, your WHERE clause needs to filter out the vast majority of those 8 million records.

Spot on! You hit on one of the most critical rules of SOQL optimization.

Even if you build a perfect Custom Index, the Salesforce query optimizer will completely ignore it and do a full table scan if you use negative operators (like !=, NOT LIKE, EXCLUDES) or leading wildcards (like LIKE '%something'). The database engine simply cannot use an index efficiently for those types of searches.

To close the loop on the selectivity threshold: for the optimizer to actually use your custom index on those 8 million records, your WHERE clause generally needs to target less than 5% of the total records (and usually under a hard cap of 333,333 records). If it returns more than that, the optimizer decides it is faster to just scan the whole table anyway.


## You must perform cross object filtering (Account → Case → Product__c) in one SOQL like query . How will you do it in Apex ?

```
List<Account> targetedAccounts = [
    SELECT Id, Name 
    FROM Account 
    WHERE Id IN (SELECT AccountId FROM Case WHERE Product__c = 'Specific Product')
];
```


