# Ticket Printer
This project is an automated solution to print tickets and items as they get
assigned. It is a library that exposes object standards and a server for making
new ticket sources (github, jira, trello, etc...) and printers easy to connect to
each other.

## Installation
This is a Node.js project, and requires npm and node to build the project.  
First clone or download the repository from github, and then run `npm install` in
the root directory to install all the dependencies.

## Building, Testing, and other Scripts
This project contains several npm scripts to help build and test the project.  
- `npm run build`: builds the project into `dist/ticket-printer.js`, happens
after `npm install` by default.  
- `npm test`: runs mocha tests on the project
- `npm test:ci`: runs mocha tests and returns a report to be read by circleci, happens
after making a PR or new branch on github.
- `npm test:debug`: runs mocha tests with a debugger that can be inspected on port 5858.
You can use a node-debugger (such as atom's `Node Debugger`) to attach and inspect the
process.

## Running
This project has no executable, but it has example scripts in the `example_scripts`
directory, which show basic use cases and demonstrate how to use the bundled objects.

## System Design
This project uses a combination of `watches`, `hooks`, and `printers` to get
and print tickets at either a time interval, or on tiggered events.

### `ActivityWatcher`
The `ActivityWatcher` is the server that collects `watches`, `hooks`, and `printers`
and acts as a mediator. `watches` and `hooks` do not need to know how their tickets 
will be printed, and `printers` do not need to know how to get new tickets, or who 
to get them from.

#### `#constructor([environment])`
Builds the `ActivityWatcher` object, and can take in an environment variable for
specific settings when running.

```javascript
var ActivityWatcher = require('ticket-printer').ActivityWatcher;
var aw = new ActivityWatcher({printLogs:true});
```

#### `#addPrinter(printer)`
Adds a printer object for watches and hooks to print to. You can use a bundled printer
or you can write your own printer (look at `printers` section).

```javascript
var ActivityWatcher = require('ticket-printer').ActivityWatcher;
var consolePrinter = require('ticket-printer').consolePrinter;

var aw = new ActivityWatcher();
aw.addPrinter(consolePrinter);
```

#### `#addWatch(watch, interval)`
Adds a watch object for printing tickets at an intervals. This is useful if you can not
add your own hooks to a project or organization. The interval is an integer in ms to check
for new tickets from the watch. You can use a bundled watch or you can write your own
watch (look at `watches` section).

```javascript
var ActivityWatcher = require('ticket-printer').ActivityWatcher;
var timeWatch = require('ticket-printer').timeWatch;

var aw = new ActivityWatcher();
aw.addWatch(timeWatch, 1000);
```

#### `#addHook(hook)`
Adds a hook object for printing tickets when an event occurs.  
**TODO: This has yet to be defined**

#### `#reset()`
Stops watches from running and removes all watches, hooks, and printers.

```javascript
var ActivityWatcher = require('ticket-printer').ActivityWatcher;
var timeWatch = require('ticket-printer').timeWatch;

var aw = new ActivityWatcher();
aw.addWatch(timeWatch, 1000);
aw.watches; // -> [ timeWatch ]

aw.reset();
aw.watches; // -> []
```

### `tickets`
Tickets are objects which are generated by `watches` and `hooks`, and are passed on to
(possibly multiple) printers by the `ActivityWatcher`. They have the following properties:  
- `title`, a string, the title of the ticket
- `project`, a string, the project that the ticket belongs to
- `number`, a string, the id or number of the ticket
- `body`, a string, the text content of the ticket

```javascript
var exampleTicket = {
  title: "Messages are lost in queue",
  project: "Chats-R-Us",
  number: "#27a",
  body: "When sending messages using ..."
};
```

### `watches`
Watches are javascript objects which are queried for tickets at a given time interval.
This is valuable when hooks are not available (due to permissions or availability)
and manually checking via an API call would be easier. Watches need to be added to an
`ActivityWatcher`, and print to all the `printers` added to the `ActivityWatcher`.

Watches are expected to have the following properties:  
- `name`, which maps to a string.  
- `getTicketObjects`, which maps to a function that returns a list of `tickets`.
Look at the `tickets` section to see the required properties for a ticket. 

```javascript
var exampleWatch = {
  name: "My First Watch",
  getTicketObjects: function() {
    return [{
      title: "Messages are lost in queue",
      project: "Chats-R-Us",
      number: "#27a",
      body: "When sending messages using ..."
    }];
  }
};
```

### `printers`
Printers are javascript objects which can print a ticket object. They are
automatically triggered by the `ActivityWatcher` when new tickets are found.
Except for testing, it is rare that you call the functions directly.  

Printers are expected to have the following properties:  
- `name`, which maps to a string.
- `printTicket`, which maps to a function that takes a ticket object, and a watch  
object.

```javascript
var examplePrinter = {
  name: "My First Printer",
  printTicket: function(ticket, watch) {
    console.log("Found a ticket from " + watch.name);
    console.log(ticket.name + ": " + ticket.body);
  }
};
```

### `hooks`
**TODO: This has yet to be defined**

## Bundled Objects
With this project, there are several bundled examples to make understanding and
working with this project easier.

### `timeWatch`
This is a bundled watch that always returns a single ticket with the current time.

### `consolePrinter`
This is a bundled printer that prints the ticket to the console.

## Contributing
If you would like to contribute to this project, feel free to fork this repository
and make a Pull-Request. PRs should include new tests and documentation updates.
