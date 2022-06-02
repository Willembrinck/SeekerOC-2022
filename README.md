# SeekerOC-2022

SeekerOC: Prototype Scriptable Time-Traveling Queryable Debugger, with experimental object-centric TTQs, for IWST 2022.

Compatible with Pharo 9.0, Moose Suite 9.0, Pharo 10 at current date (2022-06-02).

## Baseline

Do this:
```Smalltalk
Metacello new
    baseline: 'Seeker';
    repository: 'github://Willembrinck/SeekerOC-2022:main';
    load.
```
## Main project repository

SeekerOC is a fixed version and subpart of our main project SeekerDebugger, made specifically for IWST 2022.
The updated version of our time-traveling debugger project can be found in its dedicated [repository](https://github.com/maxwills/SeekerDebugger).

## Time-Traveling Queries Usage / Quick reference:
The Quick Reference pdf document is included in the repository, and can be accessed [here](./Resources/TTQs-QuickReference.pdf).

## User Defined Queries

Developers can use the scripting area to write their own program queries.

### The Query Notation

The Query notation is a general purpose notation to write queries over collections (Any collection, not just the ones related to executions). It uses standandar selection and collection semantics, however, the only difference is that selection and collection are lazily evaluated (This should be of no concern when writing the queries. 
This is only mentioned here, because it is the factor that makes possible to query the execution like this (ie, an eager evaluation select and collect would not work)).

**Example**

```Smalltalk
"In the scripting presenter, paste the following code:"

"This query obtains an OrderedCollection containing the list of all the methods of any step that corresponds to a message send to any method with the selector #add:".

(Query from: seeker newProgramStates "or just use the workspace variable: programStates"
    select: [ :state | state isMessageSend and: [ state node selector = #add: ] ]
    collect: [ :state | state methodAboutToExecute ]) asOrderedCollection
```

Then, select all the code, and **inspect it** (right click, and select **Inspect it** from the context menu, or simply press **cmd+i**). 
You should see an inspector with the collection of the results.

## User Time-Traveling Query.

Time-Traveling Queries are just a specific type of Query. To explain how to write one, we will start from the more generic Query form (as described in the previous point).

To transform a Query into a Time-Traveling Query (with integration in the UI)

1. Use **UserTTQ** instead of the **Query** class.
2. Use **Autotype** for collected items.
3. Include the **#bytecodeIndex** key as in this example:


```Smalltalk
"In the scripting presenter, paste the following code:"
| autoResultType |
    autoResultType := AutoType new.
    (UserTTQ from: seeker newProgramStates
        select: [ :state | state isMessageSend and: [ state node selector = #add: ] ]
        collect: [ :state | 
            autoResultType newWith
            bytecodeIndex: state bytecodeIndex;
            methodClass: state methodAboutToExecute methodClass name;
            messageSelector: state methodAboutToExecute selector;
            newColumnASD: 123; "you can add any column you want like this"
            endWith ]) showInSeeker
```
Then select all the code, and **do it** (right click, and select **Do it** from the context menu, or simply press **cmd+d**). 
The query should be displayed in the query tab of Seeker (you need to manually change to the tab at the moment).

### Queries (and TTQ) Composition.

Queries and TTQs can be composed. Ie, they can be used as a data source for other queries.

```Smalltalk

| query1 query2 |
   query1 := (Query from: seeker newProgramStates "or just use the workspace variable: programStates"
    select: [ :state | state isMessageSend and: [ state node selector = #add: ] ]
    collect: [ :state | state methodAboutToExecute ]).
    
    "Can be used to compose:"
    query2 := Query from: query1 select: [:state| state messageReceiver class = OrderedCollection]. 
    "Which is equivalent to:"
    query2 := query1 select: [:state| state messageReceiver class = OrderedCollection]. 
    "Finally, to trigger the query evaluation, do"
    query2 asOrderedCollection
```
In both examples, the selection conditions are applied in order, from the innermost ones (query1 selection predicate is applied first) to the outermost ones (query2 selection predicate is applied last). 
The same applies to Time-Traveling Queries (TTQs).
The methods #select: and #collect: of Queries returns new Queries objects (not the results of the query).

### Time-Traveling Queries Notes:

- The Query object instantiation doesn't trigger the production of results.
- The field bytecodeIndex is mandatory for the time-traveling mechanism. Include it always like in the example.
- AutoType automatically creates a class (and instances. The class is not registered in the system) that serves the collection function. To make time traveling queries, it is mandatory to include the bytecodeIndex field.
