# TCR 06 — Practice Bank (70 questions with model reasoning)

> **How to use this:** cover the answer block, give yourself **45 seconds**, choose an option and **write two sentences of reasoning**. Then compare. Score yourself on whether you explained the *mechanism*, not merely stated the rule.
>
> Target: 20 questions tonight, timed. That is enough to make the format automatic.

---

# PART A — SQL (Q1–25)

**Q1.** Which clause filters rows *before* grouping?
A) HAVING  B) WHERE  C) ORDER BY  D) GROUP BY

<details><summary>Answer</summary>

**B) WHERE.** *SQL evaluates WHERE before GROUP BY, so it filters individual rows before any aggregation happens. HAVING runs after grouping and is the only place aggregate functions such as COUNT(*) may be used as filter conditions.*
</details>

**Q2.** `SELECT COUNT(*) FROM t;` where `t` has 5 rows, 2 of which have a NULL in column `c`. What does `COUNT(c)` return?
A) 5  B) 3  C) 2  D) NULL

<details><summary>Answer</summary>

**B) 3.** *COUNT(column) counts only non-NULL values, so the two NULLs are excluded. COUNT(*) counts rows regardless of NULLs and would return 5 — this asymmetry is the point of the question.*
</details>

**Q3.** Which join returns all rows from the left table plus matching rows from the right?
A) INNER  B) LEFT  C) RIGHT  D) CROSS

<details><summary>Answer</summary>

**B) LEFT JOIN.** *It preserves every row of the left table and fills the right table's columns with NULL where no match exists. INNER would drop the unmatched left rows entirely.*
</details>

**Q4.** `WHERE x NOT IN (1, 2, NULL)` returns no rows. Why?
A) syntax error  B) NULL comparison yields UNKNOWN  C) NOT IN is invalid  D) it needs DISTINCT

<details><summary>Answer</summary>

**B).** *NOT IN expands to `x <> 1 AND x <> 2 AND x <> NULL`, and any comparison with NULL evaluates to UNKNOWN rather than TRUE. Since the whole conjunction can never be TRUE, no row qualifies. NOT EXISTS avoids this behaviour.*
</details>

**Q5.** Which is NOT rollback-able?
A) DELETE  B) UPDATE  C) TRUNCATE  D) INSERT

<details><summary>Answer</summary>

**C) TRUNCATE.** *TRUNCATE is a DDL statement that deallocates data pages and performs an implicit commit, so there is no undo record to roll back. DELETE is DML, logs each row removal, and can be rolled back within a transaction.*
</details>

**Q6.** A table has a composite primary key `(a, b)`, and column `c` depends only on `a`. Which normal form is violated?
A) 1NF  B) 2NF  C) 3NF  D) BCNF

<details><summary>Answer</summary>

**B) 2NF.** *2NF forbids partial dependencies — a non-key attribute depending on only part of a composite key. Here c depends on a alone rather than on the whole key (a, b), so the table must be split to reach 2NF.*
</details>

**Q7.** Which ACID property guarantees that a committed transaction survives a crash?
A) Atomicity  B) Consistency  C) Isolation  D) Durability

<details><summary>Answer</summary>

**D) Durability.** *Once a commit is acknowledged, the change is written to persistent storage via the write-ahead log, so it survives power loss or a crash. Atomicity concerns all-or-nothing execution, which is a different guarantee.*
</details>

**Q8.** Reading uncommitted data from another transaction is called:
A) phantom read  B) dirty read  C) non-repeatable read  D) lost update

<details><summary>Answer</summary>

**B) dirty read.** *A dirty read observes data another transaction has written but not committed, so the value may be rolled back and never actually existed. READ COMMITTED and stricter levels prevent it.*
</details>

**Q9.** With salaries 100, 100, 90, what does `RANK()` produce, ordered descending?
A) 1,1,2  B) 1,2,3  C) 1,1,3  D) 1,1,1

<details><summary>Answer</summary>

**C) 1,1,3.** *RANK assigns the same rank to ties and then skips the intervening numbers, so the third row is 3, not 2. DENSE_RANK would give 1,1,2 — which is why DENSE_RANK is the right choice for "Nth highest distinct value" queries.*
</details>

**Q10.** Which prevents an index from being used?
A) `WHERE id = 5`  B) `WHERE name LIKE 'a%'`  C) `WHERE YEAR(date) = 2023`  D) `WHERE id > 10`

<details><summary>Answer</summary>

**C).** *Wrapping the indexed column in a function means the stored index keys no longer match what is being compared, so the engine must scan and evaluate the function per row. Rewriting it as a date range `date >= '2023-01-01' AND date < '2024-01-01'` restores index usage.*
</details>

**Q11.** Difference between UNION and UNION ALL?
A) none  B) UNION removes duplicates  C) UNION ALL sorts  D) UNION is faster

<details><summary>Answer</summary>

**B).** *UNION eliminates duplicate rows, which requires an internal sort or hash and therefore costs more. UNION ALL simply concatenates the result sets and is faster, so it is preferred whenever duplicates are impossible or acceptable.*
</details>

**Q12.** A primary key differs from a unique key because it:
A) allows NULLs  B) is NOT NULL and one per table  C) is faster  D) cannot be composite

<details><summary>Answer</summary>

**B).** *A primary key must be NOT NULL and a table can have only one, since it is the canonical row identifier used by foreign keys. A unique key also enforces uniqueness but permits NULLs, and a table may have several.*
</details>

**Q13.** `SELECT d.name, COUNT(*) FROM Dept d LEFT JOIN Emp e ON ... GROUP BY d.name` returns 1 for an empty department. Fix?
A) use INNER JOIN  B) use COUNT(e.id)  C) add HAVING  D) use DISTINCT

<details><summary>Answer</summary>

**B) COUNT(e.id).** *The LEFT JOIN emits one row with NULL employee columns for a department with no employees, and COUNT(*) counts that row. COUNT(e.id) ignores NULLs and therefore correctly returns 0.*
</details>

**Q14.** Which is a DDL command?
A) INSERT  B) UPDATE  C) ALTER  D) SELECT

<details><summary>Answer</summary>

**C) ALTER.** *DDL defines or modifies schema structure — CREATE, ALTER, DROP, TRUNCATE — and carries an implicit commit. INSERT, UPDATE and SELECT operate on the data itself and are DML.*
</details>

**Q15.** A correlated subquery differs from a normal subquery because it:
A) is faster  B) references the outer query  C) returns one row  D) must use EXISTS

<details><summary>Answer</summary>

**B).** *A correlated subquery references a column from the outer query, so conceptually it is re-evaluated for each outer row rather than executed once. This makes it more expressive but typically more expensive, though optimisers often rewrite it as a join.*
</details>

**Q16.** Purpose of an index?
A) enforce constraints  B) speed up data retrieval  C) save disk space  D) speed up inserts

<details><summary>Answer</summary>

**B).** *An index is a B+ tree structure that lets the engine locate rows without a full table scan, turning roughly O(n) lookups into O(log n). The trade-off is extra storage and slower writes, since every insert, update and delete must maintain the index.*
</details>

**Q17.** Second-highest salary. Which is correct?
A) `SELECT MAX(sal) FROM emp`
B) `SELECT MAX(sal) FROM emp WHERE sal < (SELECT MAX(sal) FROM emp)`
C) `SELECT sal FROM emp ORDER BY sal DESC LIMIT 1`
D) `SELECT MIN(sal) FROM emp`

<details><summary>Answer</summary>

**B).** *The inner query finds the overall maximum and the outer query takes the largest salary strictly below it, which is by definition the second-highest distinct value. Options A and C both return the highest salary, and D returns the lowest.*
</details>

**Q18.** In a transaction, which command undoes all changes since the last commit?
A) COMMIT  B) ROLLBACK  C) SAVEPOINT  D) DELETE

<details><summary>Answer</summary>

**B) ROLLBACK.** *ROLLBACK discards every uncommitted change, restoring the database to its state at the last commit. This is the mechanism that implements the Atomicity guarantee of ACID.*
</details>

**Q19.** Which isolation level permits phantom reads but prevents non-repeatable reads?
A) READ UNCOMMITTED  B) READ COMMITTED  C) REPEATABLE READ  D) SERIALIZABLE

<details><summary>Answer</summary>

**C) REPEATABLE READ.** *It guarantees that rows already read cannot change if re-read, but does not by itself prevent new rows appearing that match the same predicate. Only SERIALIZABLE eliminates phantoms in the standard.*
</details>

**Q20.** A view is:
A) a copy of a table  B) a stored query treated as a virtual table  C) an index  D) a temporary table

<details><summary>Answer</summary>

**B).** *A view stores the query definition, not the data, and is re-evaluated against the base tables on each access. This gives abstraction and column-level security, whereas a materialised view does store results and must be refreshed.*
</details>

**Q21.** `DELETE FROM emp;` with no WHERE clause:
A) errors  B) deletes all rows  C) deletes nothing  D) drops the table

<details><summary>Answer</summary>

**B).** *Without a WHERE clause the predicate matches every row, so the entire table's contents are removed while the structure remains. Unlike TRUNCATE this is logged row by row and can be rolled back inside a transaction.*
</details>

**Q22.** Best defence against SQL injection?
A) escaping quotes  B) parameterised queries  C) hiding error messages  D) input length limits

<details><summary>Answer</summary>

**B) parameterised queries.** *A prepared statement sends the SQL structure and the user data over separate channels, so input is always treated as a value and can never be parsed as SQL syntax. Manual escaping is error-prone and fails against encoding edge cases.*
</details>

**Q23.** Which returns rows in the first result set but not the second?
A) UNION  B) INTERSECT  C) EXCEPT/MINUS  D) JOIN

<details><summary>Answer</summary>

**C) EXCEPT (MINUS in Oracle).** *It performs set difference, returning rows present in the first query and absent from the second. INTERSECT returns only the rows common to both.*
</details>

**Q24.** In `SELECT a, COUNT(*) FROM t GROUP BY a HAVING COUNT(*) > 1`, what does the result represent?
A) all rows  B) distinct values of a  C) duplicate values of a  D) NULL values

<details><summary>Answer</summary>

**C) duplicate values of a.** *Grouping collapses identical values of `a` into one row each and counts them; HAVING then keeps only groups whose count exceeds one, which is precisely the set of duplicated values. This is the standard idiom for finding duplicates.*
</details>

**Q25.** Denormalisation is chosen primarily to:
A) save space  B) improve read performance  C) enforce integrity  D) reduce redundancy

<details><summary>Answer</summary>

**B).** *Denormalisation deliberately reintroduces redundancy so that frequently-run queries avoid expensive joins, which suits read-heavy analytical workloads. The cost is extra storage and the risk of inconsistency on writes, so the decision depends on the read/write ratio.*
</details>

---

# PART B — OOPs (Q26–45)

**Q26.** Which achieves run-time polymorphism in C++?
A) function overloading  B) operator overloading  C) virtual functions  D) templates

<details><summary>Answer</summary>

**C) virtual functions.** *A virtual function is dispatched through the object's vtable at run time using its actual type, so a base-class pointer invokes the derived override. Overloading and templates are resolved by the compiler and are therefore compile-time polymorphism.*
</details>

**Q27.** Two functions differing ONLY in return type are:
A) overloaded  B) overridden  C) a compile error  D) templates

<details><summary>Answer</summary>

**C) a compile error.** *Overload resolution uses the parameter list, not the return type, so the compiler could not tell the two apart at a call site. Overloading requires a difference in the number, types or order of parameters.*
</details>

**Q28.** Why must a base class destructor be virtual?
A) performance  B) so the derived destructor runs on `delete basePtr`  C) required by C++  D) to allow inheritance

<details><summary>Answer</summary>

**B).** *Deleting a derived object through a base pointer with a non-virtual destructor invokes only the base destructor, leaking whatever the derived part owns. Making it virtual causes dynamic dispatch, so the derived destructor runs first and the base's afterwards.*
</details>

**Q29.** An abstract class in C++ is one that:
A) has no members  B) has at least one pure virtual function  C) cannot be inherited  D) has only static members

<details><summary>Answer</summary>

**B).** *Declaring any function `= 0` makes the class abstract and prevents instantiation, because part of its interface has no implementation. Pointers and references to it remain legal and are how polymorphism is used.*
</details>

**Q30.** Assigning a derived object to a base object by value causes:
A) a compile error  B) object slicing  C) a memory leak  D) polymorphism

<details><summary>Answer</summary>

**B) object slicing.** *Only the base sub-object is copied, so derived members are discarded and the copy has no polymorphic behaviour. Passing by reference or pointer avoids the copy and preserves dynamic dispatch.*
</details>

**Q31.** Encapsulation differs from abstraction because encapsulation:
A) hides complexity  B) hides data via access modifiers  C) enables inheritance  D) is compile-time

<details><summary>Answer</summary>

**B).** *Encapsulation is the mechanism of bundling data with its methods and restricting access through private and protected specifiers. Abstraction is the design goal of exposing only essential behaviour, and encapsulation is one of the means by which it is achieved.*
</details>

**Q32.** Constructor execution order for `class C : public B` and `class B : public A`?
A) C, B, A  B) A, B, C  C) B, A, C  D) undefined

<details><summary>Answer</summary>

**B) A, B, C.** *Construction proceeds from the most-base class downward, because a derived constructor may depend on its base being fully initialised. Destruction runs in the exact reverse order: C, then B, then A.*
</details>

**Q33.** The Rule of Three says that if a class needs a destructor it likely also needs:
A) a constructor  B) a copy constructor and copy assignment operator  C) virtual functions  D) a friend function

<details><summary>Answer</summary>

**B).** *Needing a destructor signals that the class manages a raw resource, so the compiler-generated shallow copy would leave two objects owning the same memory and cause a double free. Correct copy semantics therefore require both a copy constructor and a copy assignment operator.*
</details>

**Q34.** Which SOLID principle does "add a subclass instead of editing a switch statement" satisfy?
A) Single Responsibility  B) Open/Closed  C) Liskov  D) Interface Segregation

<details><summary>Answer</summary>

**B) Open/Closed.** *The system becomes open to extension through new subclasses while existing, already-tested code stays closed to modification. Editing a switch statement for every new type risks regressions in code that previously worked.*
</details>

**Q35.** Can a constructor be virtual in C++?
A) yes  B) no, the vptr is not yet established  C) only in abstract classes  D) only with templates

<details><summary>Answer</summary>

**B).** *Virtual dispatch works through the vtable pointer, which the constructor itself is responsible for setting up, so no dispatch mechanism exists before construction completes. Destructors run when the object fully exists and therefore can be virtual.*
</details>

**Q36.** Composition is preferred over inheritance mainly because it:
A) is faster  B) avoids tight coupling and is changeable at run time  C) uses less memory  D) supports polymorphism

<details><summary>Answer</summary>

**B).** *Inheritance binds a subclass to the base class's implementation permanently at compile time, so base changes can break subclasses. Composition delegates to an object that can be swapped at run time, keeping modules loosely coupled.*
</details>

**Q37.** The diamond problem is solved in C++ by:
A) interfaces  B) virtual inheritance  C) templates  D) friend classes

<details><summary>Answer</summary>

**B) virtual inheritance.** *Declaring the intermediate classes as `virtual public Base` makes them share a single base sub-object instead of each holding its own copy, removing the ambiguity when accessing inherited members. Java avoids the problem entirely by disallowing multiple class inheritance.*
</details>

**Q38.** A `static` member function cannot:
A) be public  B) access non-static members  C) be called  D) be overloaded

<details><summary>Answer</summary>

**B).** *A static function belongs to the class rather than to any instance, so it has no `this` pointer and no object whose non-static members it could reference. It may access other static members freely.*
</details>

**Q39.** Overriding requires:
A) different parameters  B) the same signature plus inheritance  C) different return types  D) the static keyword

<details><summary>Answer</summary>

**B).** *An override must match the base method's signature exactly, with only covariant return types permitted, and must appear in a derived class. If the parameters differ it becomes a new hiding function rather than an override — which is why the `override` keyword is valuable, since the compiler then rejects the mistake.*
</details>

**Q40.** The Liskov Substitution Principle is violated when:
A) a subclass adds methods  B) a subclass breaks behaviour the base promised  C) there are too many classes  D) inheritance is deep

<details><summary>Answer</summary>

**B).** *LSP requires a subtype to be usable anywhere its base type is expected without surprising the caller. The classic violation is `Square extends Rectangle`, where setting the width also alters the height and breaks code that assumed the dimensions vary independently.*
</details>

**Q41.** In C++, `struct` differs from `class` in that:
A) struct cannot have methods  B) struct members default to public  C) struct cannot inherit  D) there is no difference at all

<details><summary>Answer</summary>

**B).** *The only language-level difference is the default access level — public for `struct`, private for `class`, which also applies to the default inheritance mode. Structs may otherwise have constructors, methods, inheritance and access specifiers exactly like classes.*
</details>

**Q42.** The Singleton pattern guarantees:
A) thread safety  B) exactly one instance with a global access point  C) fast object creation  D) immutability

<details><summary>Answer</summary>

**B).** *A private constructor plus a static accessor ensures only one instance can exist and provides a single path to it. Thread safety is not automatic — it must be arranged, for example with a function-local static in C++11 — and the pattern is criticised as disguised global state.*
</details>

**Q43.** A pure virtual function is declared as:
A) `virtual void f();`  B) `virtual void f() = 0;`  C) `static void f();`  D) `void f() override;`

<details><summary>Answer</summary>

**B).** *The `= 0` specifier marks the function as having no base implementation, making the class abstract and requiring every concrete derived class to supply an override. This is how C++ expresses an interface contract.*
</details>

**Q44.** "High cohesion, low coupling" means:
A) many small classes  B) each module has one clear purpose and few dependencies  C) using inheritance heavily  D) avoiding interfaces

<details><summary>Answer</summary>

**B).** *High cohesion means everything inside a module serves a single well-defined responsibility, making it easy to understand and change. Low coupling means modules depend on one another minimally, so a change in one does not ripple through the system.*
</details>

**Q45.** `friend` in C++ is used to:
A) inherit privately  B) grant an external function access to private members  C) create a copy  D) enable polymorphism

<details><summary>Answer</summary>

**B).** *A friend declaration lets a specific non-member function or class read private and protected members. It deliberately relaxes encapsulation and is justified mainly when the alternative is worse — for instance overloading `operator<<`, whose left operand is an ostream and therefore cannot be a member function.*
</details>

---

# PART C — FSD & Web (Q46–60)

**Q46.** Which HTTP method is idempotent but not safe?
A) GET  B) POST  C) PUT  D) OPTIONS

<details><summary>Answer</summary>

**C) PUT.** *PUT modifies server state, so it is not safe, but it replaces the resource with the complete state supplied — repeating it leaves the resource identical, which is idempotency. POST creates a new subordinate resource each time and is therefore neither.*
</details>

**Q47.** A user is logged in but tries to access an admin page and is refused. Correct status code?
A) 400  B) 401  C) 403  D) 404

<details><summary>Answer</summary>

**C) 403 Forbidden.** *The user is authenticated, so identity is not the issue; the server understands the request but the user lacks permission. 401 would be correct only if credentials were missing or invalid.*
</details>

**Q48.** What does `console.log(1); setTimeout(()=>console.log(2),0); Promise.resolve().then(()=>console.log(3)); console.log(4);` print?
A) 1 2 3 4  B) 1 4 3 2  C) 1 4 2 3  D) 1 3 4 2

<details><summary>Answer</summary>

**B) 1 4 3 2.** *Synchronous statements run first, giving 1 and 4. The event loop then drains the entire microtask queue, where the promise callback sits, before taking anything from the macrotask queue where setTimeout waits — so 3 precedes 2 even with a zero delay.*
</details>

**Q49.** Why is a JWT payload not a safe place for secrets?
A) it is too small  B) it is base64-encoded, not encrypted  C) it expires  D) it is hashed

<details><summary>Answer</summary>

**B).** *The payload is base64url-encoded and therefore trivially decoded by anyone holding the token. The signature guarantees integrity — that the contents have not been altered — but provides no confidentiality.*
</details>

**Q50.** Using an array index as a React `key` breaks when:
A) the list is static  B) items are reordered or filtered  C) the list is short  D) never

<details><summary>Answer</summary>

**B).** *Keys give React a stable identity during reconciliation, and an index is positional rather than identity-based. When the list is reordered or an item is inserted, the same key maps to a different item, so React reuses the wrong DOM nodes and component state attaches to the wrong row.*
</details>

**Q51.** REST requires statelessness primarily because it:
A) reduces code  B) lets any server instance handle any request  C) improves security  D) simplifies the database

<details><summary>Answer</summary>

**B).** *If no session is held on a particular server, every request carries its own context and can be routed to any instance behind a load balancer. That property is exactly what makes horizontal scaling possible.*
</details>

**Q52.** Node.js performs poorly on CPU-bound tasks because:
A) it is interpreted  B) a heavy computation blocks the single-threaded event loop  C) it lacks libraries  D) it uses too much memory

<details><summary>Answer</summary>

**B).** *JavaScript executes on one thread, so a long synchronous computation occupies it and no other callbacks can be processed, stalling every concurrent request. The remedies are worker threads, clustering, or moving the work to another service.*
</details>

**Q53.** A "CORS error" in the browser is fixed by:
A) changing the frontend fetch call  B) the server sending correct Access-Control-Allow-Origin headers  C) disabling JavaScript  D) using HTTP instead of HTTPS

<details><summary>Answer</summary>

**B).** *The browser enforces the same-origin policy and only relaxes it when the server explicitly authorises the requesting origin in its response headers. The client cannot grant itself permission, so the fix belongs on the server.*
</details>

**Q54.** `const arr = [1,2]; arr.push(3);` — what happens?
A) TypeError  B) it works; `const` prevents reassignment, not mutation  C) it creates a new array  D) compile error

<details><summary>Answer</summary>

**B).** *`const` makes the binding immutable, so `arr = somethingElse` would throw, but the object it references remains mutable. Use `Object.freeze` if genuine immutability of the contents is required.*
</details>

**Q55.** In Express, an error-handling middleware is recognised by:
A) its name  B) having four parameters `(err, req, res, next)`  C) being registered first  D) returning a promise

<details><summary>Answer</summary>

**B).** *Express inspects the function's arity, treating a four-parameter middleware as an error handler. It must be registered last, after all routes, so that errors passed to `next(err)` propagate down to it.*
</details>

**Q56.** Passwords should be stored:
A) encrypted  B) in plain text  C) hashed with a salt using a slow algorithm  D) base64-encoded

<details><summary>Answer</summary>

**C).** *Hashing is one-way, so a database breach does not reveal the original passwords, and a per-user salt defeats precomputed rainbow tables. A deliberately slow algorithm such as bcrypt or argon2 makes brute-forcing expensive, whereas encryption is reversible and base64 is merely an encoding.*
</details>

**Q57.** The virtual DOM improves performance because:
A) it is written in C++  B) it computes the minimal set of real DOM mutations  C) it caches network requests  D) it removes JavaScript

<details><summary>Answer</summary>

**B).** *Real DOM operations trigger layout and repaint and are the expensive part of rendering. React diffs the new virtual tree against the previous one and applies only the differences, batching them into as few real mutations as possible.*
</details>

**Q58.** Which correctly distinguishes authentication from authorization?
A) they are the same  B) authentication verifies identity, authorization verifies permission  C) authorization comes first  D) authentication is only for admins

<details><summary>Answer</summary>

**B).** *Authentication answers "who are you?" by validating credentials, while authorization answers "what may you do?" by checking permissions against that established identity. Authentication necessarily comes first, which is also why 401 precedes 403 in the status-code model.*
</details>

**Q59.** A `useEffect` with `[]` as its dependency array runs:
A) after every render  B) once, after the first render  C) never  D) only on unmount

<details><summary>Answer</summary>

**B).** *An empty array means the effect has no dependencies that could change, so React runs it once after mounting. Its returned cleanup function then runs on unmount, which is the standard place to clear timers or subscriptions.*
</details>

**Q60.** SQL is preferred over NoSQL when the application needs:
A) horizontal scaling  B) a flexible schema  C) complex relationships and strong ACID transactions  D) very high write throughput

<details><summary>Answer</summary>

**C).** *Relational databases enforce a fixed schema with foreign keys and provide multi-row ACID transactions, which is what financial and inventory workloads require for correctness. NoSQL trades some of those guarantees for schema flexibility and easier horizontal scaling.*
</details>

---

# PART D — DevOps (Q61–70)

**Q61.** What is a Jenkins Pipeline?
A) Container  B) Repository  C) CI/CD workflow  D) VM

<details><summary>Answer</summary>

**C) CI/CD workflow.** *A Jenkins Pipeline is a set of automated stages defined as code in a Jenkinsfile that build, test and deploy an application, representing the continuous integration and delivery workflow. A container or VM is merely where a stage might execute, and a repository only stores the source code.*

*(This is the exact sample question printed in the official instruction PDF.)*
</details>

**Q62.** A Kubernetes app is healthy internally but unreachable externally; pods are Running and passing readiness checks. Investigate first?
A) Service and Ingress configuration  B) Delete and recreate all pods  C) Upgrade the cluster version  D) Increase node CPU

<details><summary>Answer</summary>

**A) Service and Ingress configuration.** *Passing readiness probes confirms the pods are healthy and serving internal traffic, so the fault lies in the external exposure path. That path is defined by the Service — a default ClusterIP is internal-only — and by the Ingress rules that route external requests, whereas recreating pods or adding CPU does not touch the networking layer.*

*(Also from the official sample.)*
</details>

**Q63.** The main difference between a container and a virtual machine is that a container:
A) is written in Go  B) shares the host kernel and virtualises only the OS  C) cannot be networked  D) requires more memory

<details><summary>Answer</summary>

**B).** *A container isolates processes while sharing the host kernel, so it starts in seconds and occupies megabytes. A VM virtualises hardware and boots a complete guest operating system, giving stronger isolation at a far higher cost in size and start-up time.*
</details>

**Q64.** The smallest deployable unit in Kubernetes is:
A) container  B) Pod  C) Node  D) Deployment

<details><summary>Answer</summary>

**B) Pod.** *Kubernetes schedules pods, not individual containers; a pod wraps one or more tightly-coupled containers that share a network namespace and storage. Deployments and ReplicaSets are higher-level controllers that manage pods.*
</details>

**Q65.** A liveness probe failure causes Kubernetes to:
A) remove the pod from the Service  B) restart the container  C) scale up  D) log a warning only

<details><summary>Answer</summary>

**B) restart the container.** *Liveness answers "is this process still alive?", so a failure means the container is stuck and restarting it is the remedy. A readiness failure instead removes the pod from the Service endpoints without restarting it, since the container may simply still be warming up.*
</details>

**Q66.** Continuous Deployment differs from Continuous Delivery in that it:
A) skips testing  B) releases to production automatically with no manual approval  C) uses containers  D) requires Jenkins

<details><summary>Answer</summary>

**B).** *Continuous Delivery keeps every passing build in a releasable state but leaves a human gate before production. Continuous Deployment removes that gate, so any change passing the pipeline ships automatically — which demands much stronger automated test coverage and monitoring.*
</details>

**Q67.** `git rebase` should not be used on commits that:
A) are local  B) have already been pushed and pulled by others  C) are small  D) are on a feature branch

<details><summary>Answer</summary>

**B).** *Rebasing creates new commit objects with different hashes, so shared history diverges and collaborators are forced into painful reconciliation. Merge is the safe option on shared branches; rebase belongs on private, not-yet-published work.*
</details>

**Q68.** A Kubernetes Service shows no endpoints. Most likely cause?
A) the cluster is down  B) the Service selector labels do not match the pod labels  C) insufficient CPU  D) the image is missing

<details><summary>Answer</summary>

**B).** *A Service builds its endpoint list by selecting pods whose labels match its selector, so a mismatch yields an empty set and all traffic fails despite healthy pods. Checking `kubectl describe svc` for empty endpoints is the standard diagnostic for this.*
</details>

**Q69.** Why does a Dockerfile copy `package.json` before the application source?
A) alphabetical order  B) to exploit layer caching so dependency installation is reused  C) it is mandatory  D) for security

<details><summary>Answer</summary>

**B).** *Docker caches each instruction as a layer and invalidates everything after the first change. Copying the manifest and installing dependencies before the source means that editing application code reuses the expensive install layer instead of repeating it.*
</details>

**Q70.** A pod is in `CrashLoopBackOff`. The most useful next command is:
A) `kubectl get nodes`  B) `kubectl logs <pod> --previous`  C) `kubectl scale`  D) `kubectl delete namespace`

<details><summary>Answer</summary>

**B).** *CrashLoopBackOff means the container repeatedly starts and exits, so the failure is inside the application and the evidence is in the terminated instance's output. `--previous` retrieves logs from the crashed container rather than the one currently restarting.*
</details>

---

## Score yourself

| Correct out of 70 | Where you stand |
|---|---|
| 60+ | Excellent. Focus tonight on *writing* reasoning quickly, not on more content. |
| 45–59 | Solid. Re-read the topic files for the sections you missed. |
| 30–44 | Revise [01_SQL.md](01_SQL.md) and [04_DEVOPS.md](04_DEVOPS.md) — they hold the densest, most learnable marks. |
| under 30 | Work through the four topic files once more, then return here. Do not skip the reasoning writing. |

**More important than your score:** for every question you got right, did your written reasoning explain the **mechanism**? If it only asserted the rule, that is where tomorrow's marks leak away. Re-read [05_REASONING_PLAYBOOK.md](05_REASONING_PLAYBOOK.md).

Next: [../CHEATSHEET_FINAL_REVISION.md](../CHEATSHEET_FINAL_REVISION.md)
