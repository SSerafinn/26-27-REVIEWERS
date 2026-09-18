# ITE 116 – Integrative and Programming Technologies
# Midterm Reviewer

**Coverage:** Integrative programming foundations through your first Laravel application.
**Format:** Written examination, multiple choice.

> **How to use this reviewer.** This is a guide to the *concepts* you are responsible for — not a list of answers. Most of the exam asks you to **apply, diagnose, and judge**, not to recall definitions. So for every topic below, don't stop at "I can define this." Ask yourself: *Could I use it? Could I explain why something broke? Could I defend one approach over another?*
>
> A good test of readiness: read each **"You should be able to"** line and try to answer it out loud, from memory, before checking your notes.

---

## Contents

1. [Front-End and Back-End Integration](#1-front-end-and-back-end-integration)
2. [Server-Side vs Client-Side Programming](#2-server-side-vs-client-side-programming)
3. [PHP Fundamentals](#3-php-fundamentals)
4. [Data Exchange Formats (JSON and XML)](#4-data-exchange-formats-json-and-xml)
5. [Interoperability Between Programming Languages](#5-interoperability-between-programming-languages)
6. [Laravel Foundations: MVC, Setup, and Filesystem](#6-laravel-foundations-mvc-setup-and-filesystem)
7. [Laravel CRUD Application Development](#7-laravel-crud-application-development)
8. [Four Ideas That Run Through Everything](#8-four-ideas-that-run-through-everything)
9. [Quick Reference Tables](#9-quick-reference-tables)
10. [Self-Test](#10-self-test)

**Where the weight sits:** the heaviest coverage is on **PHP fundamentals** and **Laravel CRUD development**, followed by client/server reasoning and data formats. Interoperability is the lightest. Study accordingly, but don't skip anything.

---

## 1. Front-End and Back-End Integration

### What to know

- The **three tiers** and the one job each has: presentation (what the user sees), application (logic and decisions), data (persistent storage).
- Which technologies belong to which tier, and why a tier should not do another tier's job.
- The **request–response cycle**: what the browser sends, what the server does with it, and what comes back.
- How a form hands data to server code — specifically, what connects a field on the page to the value the server reads.
- Why the presentation tier goes *through* the application tier to reach data, instead of reaching it directly.

### Common confusions to clear up

- Which tier "owns" the data versus which tier merely *displays* it.
- What actually travels over the network in a request (and what does not).
- What causes a submitted value to arrive missing or empty on the server.

### You should be able to

- Trace a single user action from click to rendered result, naming each stage.
- Given a small system description, say which responsibilities belong to which tier.
- Explain the consequences of letting the browser talk straight to the database.

---

## 2. Server-Side vs Client-Side Programming

### What to know

- The one question that decides everything here: **where does this code run?**
- What the user can see, read, and change — and what stays hidden.
- Why some code keeps running after a page loads while other code ran only once.
- The difference between checks that exist for **convenience** and checks that carry **authority**.
- Which kinds of work belong on each side, and why putting them on the wrong side is a problem.

### Common confusions to clear up

- "It validates, therefore it's secure." Know exactly why that reasoning fails.
- Why a value produced on the server behaves differently from one produced in the browser.
- What viewing a page's source does and does not reveal — and what that implies about storing secrets.

### You should be able to

- Given any task, decide whether it belongs on the client, the server, or both, and justify it.
- Predict what happens to an application when a user disables JavaScript.
- Explain what a bypassed client-side check proves about the server behind it.

---

## 3. PHP Fundamentals

*(Heavily weighted — review actively, by writing code rather than reading it.)*

### What to know

- **Variables and types:** how variables are written, and the basic types you'll meet.
- **Operators:** arithmetic, assignment, comparison, and logical. Know the difference between loose and strict comparison, and which one you should prefer.
- **Control flow:** `if / elseif / else`, `match`, the ternary form; when each is appropriate.
- **Loops:** `for`, `foreach`, `while` — and which one suits going through a list of records.
- **Functions:** parameters, default values, and the crucial difference between **printing** a value and **returning** one.
- **Arrays:** indexed arrays (numbered from **0**) versus associative arrays (labeled with keys), how to read each, and how to add to them.
- The **list-of-records** pattern — an indexed array of associative arrays — and why almost all web data takes that shape.
- Common built-ins for counting, joining, and inspecting arrays and strings.

### Common confusions to clear up

- Off-by-one errors: which index is the first, which is the last.
- Why two values that *look* equal may or may not compare as equal.
- Why a function that prints its result is not the same as one that returns it.
- Reading a value out of an associative array versus out of an indexed one.

### You should be able to

- Choose the right data structure for a described piece of information.
- Read a short snippet and say exactly what it outputs — or why it errors.
- Write a small function that takes input and returns a computed result.
- Loop over a list of records and pull named fields out of each.

---

## 4. Data Exchange Formats (JSON and XML)

### What to know

- Why a shared text format is needed at all — what problem serialization solves.
- **JSON:** objects, arrays, keys and values, nesting, and which value types it supports natively.
- **XML:** elements, nesting, a root element, and **attributes** — and how an attribute differs from a child element.
- Converting between PHP data and JSON text in both directions, and how to tell when the conversion failed.
- Reading an XML document in PHP, including how attributes are accessed differently from elements.
- The practical trade-offs between the two formats, and which is the sensible default for new work.

### Common confusions to clear up

- Which direction each conversion function goes.
- How a decoded structure is accessed — this differs depending on how you decoded it.
- Attribute versus element in XML (this one catches people out).
- What a failed parse returns, and why you should check for it before using the result.

### You should be able to

- Convert a small table of data into correct JSON by hand.
- Look at a JSON blob and identify every object, array, key, and value.
- Look at an XML fragment and point out the root, the elements, and the attributes.
- Diagnose why a parse produced nothing usable.

---

## 5. Interoperability Between Programming Languages

### What to know

- Why programs in different languages can't simply share data directly.
- The idea of a **language-neutral interface**, and the several forms it takes: a shared data format, a file handoff, one program invoking another, and communication over the network.
- What all those approaches have in common.
- When it makes sense to keep a capability in another language rather than rewriting it.

### Common confusions to clear up

- Thinking interoperability requires the systems to be written the same way, or to run in the same place.
- Confusing a shared *file* handoff with a live *network* exchange — know how they differ.

### You should be able to

- Given two systems that must cooperate, propose a reasonable way to connect them.
- Explain what made a cross-language exchange work.
- Judge whether rewriting a library or bridging to it is the better decision.

---

## 6. Laravel Foundations: MVC, Setup, and Filesystem

### What to know

- What a **framework** is, and how it differs from a library in terms of *who calls whom*.
- **MVC**: the three parts, the single job each has, and how they correspond to the three tiers you already know.
- The **request lifecycle** through the framework, including the checkpoint stage that requests pass through before your own code runs.
- **Convention over configuration** — what it means in practice and why teams benefit.
- Requirements and the basic setup sequence for getting a project running locally.
- The **project structure**: which folders hold routes, controllers, models, views, and database definitions; which file holds configuration and credentials; and which folder you must never edit by hand.
- What the command-line tool is for and the general shape of its commands.

### Common confusions to clear up

- Library versus framework — be able to state the distinction cleanly.
- Which folder does what (mixing these up is a common error).
- Why configuration secrets live in one specific file, and why that file is not shared.
- What it means when a request never reaches your controller.

### You should be able to

- Name the folder or file where a given piece of code belongs.
- Trace a request through the framework, naming the stages in order.
- Explain a symptom such as "my code never ran" by pointing to the responsible stage.
- Justify adopting (or not adopting) a framework for a project of a given size.

---

## 7. Laravel CRUD Application Development

*(The heaviest section — expect scenario and diagnosis questions.)*

### What to know

- The four **CRUD** operations, the HTTP method each uses, and the controller method that conventionally handles each.
- **Routes:** how a URL is bound to a controller method, why routes are given names, and how the same URL can serve different purposes under different methods.
- **Migrations:** defining table structure in code rather than by hand, and the command that applies them.
- **Models:** how a model corresponds to a table, and the property that controls which fields may be filled from user input — plus what that property is protecting against.
- **Validation:** how rules are declared, what happens automatically when they fail, and how previously entered input is preserved.
- **Views:** how data reaches a template, how values are printed safely, and the directives for looping and conditionals.
- **Form requirements:** the token every state-changing form needs, and the technique used when a form must send a method that browsers don't support.
- **Retrieving a record by its identifier** without writing a query, and what happens when the record doesn't exist.
- **Delete specifically:** why a destructive action must not be a plain link, and what protects it.
- The redirect-after-save pattern and the short-lived message that follows it.

### Common confusions to clear up

- Which HTTP methods are safe to use for actions that *change* data — and which are not.
- Why an update or delete form must carry something extra beyond the usual token.
- The difference between a missing token and a missing method override — they produce different errors.
- Why a model rejects a field you tried to save.

### Error messages worth recognizing

Be able to match a symptom to its cause. For each of the following, know *what is missing or wrong* and *which file you would open*:

- A page-expired error after submitting a form.
- A "method not allowed" error on an update or delete.
- A mass-assignment complaint naming one of your columns.
- A route-not-defined error when generating a link.
- A not-found page when opening a record by id.

### You should be able to

- Write the route, controller method, and form for any one CRUD operation.
- Given an error, name the likely cause and the file to fix.
- Explain what the framework is doing for you that you previously did by hand.
- Defend a secure implementation of a destructive action over an insecure one.

---

## 8. Four Ideas That Run Through Everything

These recur across topics, and the harder questions are usually built on them.

1. **The server is the authority.** Anything the browser holds can be seen and changed by the user. Convenience can live on the client; enforcement cannot. This applies to validation, secrets, permissions, and destructive actions alike.

2. **Separate responsibilities.** Tiers, MVC parts, and folders all exist so each piece does one job. When something takes on another's job — a view querying data, a controller building markup — that's a design error, and you should be able to name it.

3. **Agree on an interface.** Front-end and back-end, two languages, two systems, two teammates: cooperation works because both sides honor a shared, neutral agreement — a data format, a naming convention, a framework convention.

4. **Don't repeat yourself.** Repetition is where bugs breed, because a fix has to be applied everywhere and one place always gets missed. Functions, components, and shared layouts all exist to solve this.

---

## 9. Quick Reference Tables

### HTTP methods and intent

| Method | Intent | Safe to trigger by a plain link? |
|---|---|---|
| GET | retrieve / display | Yes |
| POST | submit new data | No |
| PUT | update existing data | No |
| DELETE | remove data | No |

> Know *why* the last three answer "no" — it's a favorite reasoning question.

### CRUD at a glance

| Operation | HTTP method | Conventional controller method |
|---|---|---|
| Create (show form) | GET | `create` |
| Create (save) | POST | `store` |
| Read | GET | `index` / `show` |
| Update (show form) | GET | `edit` |
| Update (save) | PUT | `update` |
| Delete | DELETE | `destroy` |

### PHP essentials to have cold

| Concept | Know this |
|---|---|
| Variables | how they're written; the basic types |
| Comparison | loose vs strict — and which to prefer |
| Arrays | indexed (from 0) vs associative (by key) |
| Loops | which loop suits a list of records |
| Functions | parameters, defaults, return vs print |
| Arrays of records | how to loop one and read named fields |

### Where things live in a project

| Holds | Folder or file |
|---|---|
| URL definitions | routes file |
| Back-end logic | controllers folder |
| Page templates | views folder |
| Table representations | models folder |
| Table structure | migrations folder |
| Configuration & credentials | environment file |
| Installed dependencies | vendor folder — **never edit** |

---

## 10. Self-Test

Answer from memory. If you hesitate, that topic needs another pass. *(These are study prompts, not exam items.)*

**Concepts**
1. Name the three tiers and give each one job. Which one never talks to the browser directly?
2. What decides whether code is "client-side" or "server-side"? Give two consequences of that difference.
3. Why is a check performed in the browser not sufficient to enforce a rule?
4. What problem do data exchange formats solve that in-memory data cannot?
5. Distinguish a library from a framework in one sentence.
6. Name the three MVC parts and map each to a tier.

**Applying**
7. You must keep several labeled details about one person in a single variable. What structure, and how do you read one detail back?
8. You need to hand a list of records from server code to a browser script. What do you produce, and how does the other side read it?
9. A page must show a value fixed at the moment the page was generated. Where does that code run?
10. You must add a new page to a Laravel app. Which files do you touch, in what order?

**Analyzing**
11. A submitted field arrives empty on the server even though the user filled it in. Give two plausible causes.
12. A parse of incoming data yields nothing usable. How would you confirm the cause?
13. Your controller code never runs and the user lands elsewhere. Which stage is responsible?
14. A form submits but the framework rejects it as expired. What's missing?
15. An update submits but the framework says the method isn't allowed. What's missing — and how does this differ from #14?

**Evaluating**
16. A classmate hides a secret value in browser code "so only the app knows it." Assess this.
17. A team debates enforcing a rule in the browser, on the server, or both. What's your position and why?
18. Argue for or against adopting a framework for a growing application — and give the strongest point for the *other* side.
19. Compare writing database queries by hand with using the framework's model layer for a student project.
20. Someone claims a confirmation prompt makes deletion secure. Evaluate that claim.

---

## Final Advice

- **Practice by doing, not reading.** For the PHP and Laravel sections, open an editor and rebuild small pieces from scratch. Recognition is not recall.
- **Learn the error messages.** Several questions describe a symptom and ask for the cause. If you've seen an error before and know which file to open, those become easy points.
- **Prepare to justify, not just identify.** For the reasoning questions, more than one option will look defensible. The right answer is the one you can *explain* — usually the one that respects server authority, separation of responsibilities, or the security default.
- **Watch the qualifiers.** Words like *most likely*, *best supported*, and *most defensible* are asking for a judgment, not a definition.

Good luck.
