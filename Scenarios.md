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



