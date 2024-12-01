# Structured data management through graphs (Placeholder name: GraphDown)

## Introduction

Imagine that no one has invented spreadsheets yet, not even the concept of tabular data is really used. Then someone invents the csv format and makes it easy to edit and process it using an app like Excel. Would it be challenging to describe this to someone? I think it would be, and this is the same challenge I face when I try to describe the idea below.

The tl;dr version of the idea is: it's a format to store any data in a hierarchical and typed way, but also in free form while being human readable and editable. It's also an app to make this interactive and conveniently edit this data, and to query, process, and visualize it.

I have a huge document about this idea, what problems it solves, for whom it's useful, what are the use cases and key benefits, etc. But it's too long and too much to digest, so I'll try to skip most of it and focus on the core idea. 

## The core idea

It has 3 parts: expressive language for describing data and their relationships, a server that processes this into a queryable graph database, and a client application that makes it all accessible.

### The language

It's designed to not conflict with markdown, so any markdown file can be extended with data in this format. It's also fairly simple I think, so simple in fact that I probably don't even need explanation just an example.

```markdown
# My Todo list

I can have any content around or even inside the graph data.

(Todo)
  -- has -> task: Buy cat food
    -- is -> state: Pending
  -- has -> task: Some other task
    -- due -> date: 2024-12-10
    <- assigned -> person: John Doe
    -- is -> state: Done
    ```markdown
    ## markdown content for the specific task
   
    It can be any language in fenced code blocks not just markdown.
    This is how we can assign additional metadata or views to the task for example using TOML or HTML.
    ```
```
#### Explanation

The "Todo" in parentheses is a start of a graph, it's essentially a root typed node with the @Todo ID. 

"has" and "is" are edges (verbs) and they have a direction (the graph is directed and can also be cyclic).

"task", "state", and "date" are types for the nodes, just like the verbs they can be anything.

"Buy cat food", "Pending" are node IDs, they have to be unique but they can have actual content. So "Buy cat food" is actually just short for @Buy_cat_food = { value: "Buy cat food" }.

Indentation and arrows define the hierarchy. For example, "Buy cat food" is a child of "Todo". Similarly, "John Doe" is assigned to a task, and that task is also linked back to him.

The content in that fenced block is just for that one specific "Some other task" node. It is possible to have all kinds of data, views, and scripts in a similar way not just for each node but for every node in a given type. 

The language has many more features, like query syntax, special nodes, json properties for nodes, syntax to use it inline, etc.

### The server

Even without additional processing, the above format provides immediate value in my opinoin. I mean, it's not yet processed in any way, but it's already useful because it's obvious at a glance what it is about, LLMs can understand it perfectly, an editor like VSCode especially with Copilot can autocomplete and manipulate it, the syntax allows us to search for specifics (for example searching for `task:` will find all the tasks), etc. 

But the server is where the magic happens, it transforms this human-readable format into a queryable graph database and serves it to the client (any client). Also it will eventually work as a linter, formatter, etc.

Everything will be self-hostable or run locally, so it's not a SaaS. Even this server will just be a wasm module that can run in a browser.

The server will take a text (markdown or anything else) content, parse it, and replicate the data in a graph database. It will also store the original content of course, and it will be able to query the data as well.  

So if we just run `graphdown serve xy.md` it will start a server, will make a db file for the data which it also updates when the text changes, and we can query it with a simple http request to get back for example all the Tasks that have Pending state attached to them.

### The client

The client will be a browser based app with so many features I'm not even sure where to start.

First of all, obviously it will have autocomplete features, syntax highlighting, and all kinds of editor features so even though we can use any verb, type, and in general any structure, we don't have to remember what we used before, we can just start typing and it will suggest the rest.   

For example, the "state" typed node with "Pending" value is nothing special, it could be anything, it could be "completed" typed node with "false" value for example. It has no meaning in itself, but because this app will suggest the verbs, types, and nodes we used before in a similar context, the graph can act as a schema. Also, verbs and types together with the connected node types allows us and for an LLM to infer everything about the data.

Let's say we add another task with this app to our Todo graph, as soon as we start a new line under Todo, it will suggest "-- has -> task: ", then we can write the name of the task and after new line and tab it will suggest "-- is -> state: Pending" and "-- is -> state: Done" both. Essentially, writing any kind of data once will make it easy to write it again following the same structure, it's like automatic forms.

This whole document is a short version of another. I can't mention everything here, you have to think about it. For example I could mention that because the graph syntax is simple, an LLM can write it for us - we can just copy-paste a work email and AI can just create a task for it. There are all kinds of similar possibilities. 

#### Views

The client app is not just a text editor. It will have the ability for example to upload images as content for the nodes but more importantly, it will have the ability to process and display special node types like views. 

Because the data is graph, we don't need separate dependency graphs and our data can easily cascade. This allows us to define views for any node or node type and to combine them in connected nodes.

What this means for example is that we can define a view for "date" typed nodes:  
`{new Date(value).toLocaleDateString('en-GB', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}`  
and when we are in view mode, we can see "Tuesday 10 December 2024" instead of "2024-12-10" for the task due date.  
The view can be anything but what I'm sure is that by default it will be in Svelte which allows us to have reactive and interactive views (just html+js with maybe tailwind).  

These views can also be used to change the data they're attached to, so we can have a date type view which not just shows a nicely formatted date but also allows us to change it with a calendar input. 

Because these views can cascade in both directions (parent nodes can incorporate their children's views, and child nodes can reference their parents' data), we can combine them to create complex interactive interfaces. For example we could also make an interactive view for the "state" typed nodes:  
`<button on:click={() => {value = value === "Pending" ? "Done" : "Pending"}}>{value === "Pending" ? "⏳" : "✅"}</button>`  
and then in the task typed node's view we can show both together with the task title.  

Just like that we have a todo app that's easily customizable and extendable with all kinds of other advantages. 

The app will come with default views for certain types, but there will likely be a marketplace for additional views, templates, and even complete 'applications'.

Again, I can't mention everything. For example I said earlier that the language has query syntax: this means that we don't have to use the data in the way we wrote it, we can make queries to get them back in a different way.  
For example our base markdown file might be a journal type thing where we write an entry for each day and then we insert a task into the todo graph inside these day entries. This means that our Tasks will be all over the place in the file but it doesn't matter because we can just make a new Todo section in the document with a query that shows our whole Todo graph in that place (queries can also have views).  
Similarly, we can have a "People I'm working with" section where we query for all the person typed nodes to display only their name and the pending tasks assigned to them.  
These queries can be complicated but it's not harder than an SQL query so most people will be able to write them.

## Important: Why Graphs and Types Matter

The Todo example has an `<- assigned -> person: John Doe` connection.  
Imagine that you also have a "Contacts" graph in the document and you list people you know in it with all kinds of details. If you first create this John Doe node in the Todo graph, then he will be suggested in the Contact graph as well or if you have a query there for all the persons, he will be there instantly.  
If you already have him in the Contacts graph, then his name will be suggested in the Todo graph as soon as you connect a person typed node to a task.  
If this person has connection anywhere in the document to anything (company, phone number, spouse, where you met, where they live, etc.) then all of these information will be available wherever the node is used.  

This person will be listed in the Todo graph but the task will be listed in the person's node as well since it's connected to him with the 'assigned' edge. This is the power of graphs. The data is connected in a natural way and it's easy to manage and query it (the connection is bidirectional in this case but that only matters with queries).

## Conclusion

I found it challenging to decide how to write this document. The problem is that there are too many possibilities. For example I used todo as example but I could have used a product catalog, a website, or a CRM just as well and with those I could have focused on different features. I really can't write about everything planned.  

I didn't even mention process nodes which essentially allows us to do pretty much everything Excel does.  
I didn't mention weighted and parameterized edges either even though they can be used to create complex relationships, for example we could have distance defined between location type nodes and then we could query for the shortest path between them.  
Or dynamically generated nodes, the ability to export the result of queries in different formats, the ability to use the generated and combined svelte views as an embeddable web component, etc.

If the idea is as good as I think it is, then I don't think monetization will be a problem. I could just follow Obsidian's example and provide this whole thing for free with a paid sync and backup service or for enterprises with a paid version.

## Why Graphs and Types Matter (again)

> This section is like 15 pages long in my original idea document so even though I mentioned it above, I also asked Claude to make a summary of it:

At first glance, you might wonder why we need typed graphs when simpler solutions like TOML in markdown could handle basic use cases like todo lists or project tracking. The answer lies in how we actually work with information.

Consider this common scenario:

-   You read an article relevant to your current project and bookmark it
-   You create a Trello card to track the article's action items
-   You want to share it with a colleague who isn't on Trello, so you search your contacts
-   They reply with a calendar invite to discuss it
-   You prepare by taking notes in Obsidian
-   You save useful code snippets from the article in GitHub Gists
-   Later, you reference this article in a document you're writing

Currently, these interconnected pieces of information live in separate tools, each with its own data format and organization system. The natural connections between them - the fact that they're all part of the same workflow - are lost. There's no way to see all related tasks, notes, contacts, bookmarks, emails, calendar events, and code snippets in one place.

This isn't just a personal productivity issue. In professional settings, the problem is often more severe. For example, an e-commerce business might juggle:

-   Product data across multiple spreadsheets
-   Customer information in a CRM
-   Inventory in an ERP system
-   Marketing content in various CMS platforms
-   Multiple language versions of the same content
-   Analytics data related to all of the above from different platforms

Connecting these data sources typically requires extensive custom development work just to create basic integrations.

GraphDown isn't just another task manager or note-taking tool - it's an attempt to solve the fundamental problem of data fragmentation. By representing information as a typed graph, it can:

-   Maintain natural connections between different types of data
-   Query across these connections to find every relevant information
-   Adapt to new types of data without requiring structural changes
-   Present the same information in different ways for different purposes

Think about your own digital life - how many connections between pieces of information are lost because they're trapped in different tools? How often do you have to manually recreate these connections, or worse, fail to make them at all because it's too cumbersome?

The power of GraphDown isn't in any individual feature - it's in its ability to maintain the natural relationships between all your digital information, regardless of what that information is or how you choose to organize it.
