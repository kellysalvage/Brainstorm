# Brainstorm

Brainstorm is a system that allows users to quickly create lists of items in markdown-like text and then transform them into useful, actionable data. The text can be defined using a brainstorm structure which is also markdown-like and defines the types that can be represented in a brainstorm document. A branstorm structure document is a brainstorm document that defines a structure.
The system provides text to data and data to text where the text and data are associated with the same brainstorm structure. 

## Markdown
Brainstorm documents and structure documents are Markdown, with the usual meanings, so they can be read and edited in any text editor. Structure documents use the file extension `.bss`, and brainstorm documents use `.bsd`.
* **Headings are sections.** `#`, `##` and so on group the items below them. In the app's tree view a section is shown like a folder, and sections are kept when a brainstorm is written back out as a document.
* **Plain text is notes.** A paragraph that is not an item, a heading or an element is kept and shown, but it is not data. This is how to write comments and explanations.
* **A line is an item only when the word before the colon is a known type or alias.** "Remember to call Bob: it is urgent" is a note. A line such as `Gaol: Book a holiday` looks like an item with a mistyped type, so it is flagged rather than silently treated as a note.
* **Text values can use inline Markdown**, such as bold, italics and links. The app shows them formatted.
* **`*` starts a list item only directly under a list element.** Anywhere else it is an ordinary bullet in a note.

### Fences in structure documents
In a structure document, types, choices and functions are only read from code blocks marked `bss`:

````
```bss
t Goal|tg
    > Something you want to achieve.
    s+! Name|Goal|A short name for the goal
```
````
Apart from the front matter at the top (see Versions and change notes), everything outside the `bss` blocks is documentation: headings, explanations, pictures and examples, shown as help in the app. This keeps a structure document readable, and it means a note can never be mistaken for a definition. If a definition contains three backticks, for example in a description, use four backticks for the fence instead.

Brainstorm documents are not fenced, because they are for everyone. In this README, examples of brainstorm documents are marked `bsd` so that tools can tell the two apart.

### Linking a document to its structure
Every brainstorm document names its structure at the very top, between two lines of three dashes. This is called front matter, and most Markdown editors understand it and keep it apart from the text. The structure is chosen when the document is created, so the editor can highlight, complete and check the text from the first line.
```bsd
---
structure: fearless-focus.bss
---
# Kelly's Fearless Focus plan
```
* **A file path** is used for a structure kept as a file. A relative path starts from the folder the document is in.
* **A web address** is used for a published program. The Brainstorm service checks that the user is allowed to use the program before it sends the structure.
* **A version** can be added to a web address with `@`, as in `https://brainstorm.app/programs/fearless-focus@3`. The document then stays on that version until the user chooses to update it. A web address without a version always gets the latest version.
* **`built-in`** uses only the built-in types, such as goals, tasks and risks, for documents that do not need a structure of their own: `structure: built-in`.
* **A document with no structure**, such as one written in another editor, is read using the built-in types, and flagged so that a structure can be added. If the user does not choose one, the app adds `structure: built-in` to the document's front matter.

### Versions and change notes
A structure that other people use, such as a published program, changes over time. Its author lists its versions in the structure document's front matter. Each version is written as its number and date, then a colon and a pipe, then the change notes on indented lines below. A note starting with `nb:` tells users something they need to change in their own documents:
```bss
---
versions:
  1 2026-10-21: |
    Initial version
  2 2026-11-01: |
    Added a Sleep measure to the daily review.
  3 2027-01-15: |
    Added a Mood rating to the weekly review.
    nb: Weight is now called BodyWeight. Write BodyWeight instead of Weight in your plan.
---
```
Front matter is written in YAML, which is what Markdown editors expect. The pipe tells YAML to keep the notes exactly as they are written, so a note can contain a colon, as `nb:` does.

The rules are:
* Versions are listed oldest first, so a new version is added at the bottom.
* Version numbers are whole numbers that go up. The highest one is the structure's current version.
* A program can only be published with a version number higher than the last one published.
* When a document's structure has a newer version, the app shows the change notes of every version in between, newest first, so a user who skipped several versions sees everything that changed.
* The app also checks the user's own document against the new version and shows exactly what would be flagged, before the user chooses to update. Authors' notes explain why; the check shows what actually breaks.

## Brainstorm Structure syntax
All declarations are made starting with a token defining what is being declared. A complex item with child elements has the child elements listed below its description, which explains what the type is for and is shown to users as help. Description lines start with `>`, so they never look like elements (see Descriptions). All elements must be pre-defined types or types that are defined in the same brainstrom document: the key at the moment is to keep it simple since this is not meant to be a turning complete programming language, just a way to easily define DDD models.

### Declaring a type
A type is declared with a line starting with t, followed by a space thn the name of the type, a pipe | and the alias. The alias is required. For a type that should always be written in full, make the alias the same as the name, as in `t Recipe|Recipe`. The lines after the declaration that start with `>` describe the type. After the description, the elements of the type are listed with one per line. Each element consists of the alias or name, followed by a space, the name of the element, an optional caption and and optional description. These elements are used for quickly generating UI representations of items and for describing items in brainstorm document hints. The use of squere brackets indicates that the part of the element definition is optional. E.G.
```
t name|alias
    > this is a description of the type for use by the system to display help and tips. 
    s!! Name|Name|Name must always be after the description.
    i IntegerPropertyName[|Caption][|InetegerPropertyDescription]
    d DateTimeProperty[|Caption]|[DateTimePropertyDescription]
    dt TimeOnlyProperty[|Caption]|[TimeOnlyPropertyDescription]
    dd DateOnlyProperty[|Caption]|[DateOnlyPropertyDescription]
    n DecimalNumberProperty[|Caption][|DecimalNumberPropertyDescription]
    s StringOrTextProperty[|Caption][|StringPropertyDescription]
    l:s ListOfStringsProperty[|Caption]|[ListOfStringsPropertyDescription]
    l:i ListOfIntegerProperty[|Caption][|ListOfIntegersPropertyDescription]
    l:t ListOfTaskProperty[|Caption][|ListOfTaskPropertyDescription]
    tx AnExtraProperty[|Caption]|[AnExtraPropertyDescription]
    MyThing AnotherExtraProperty[|Caption][|AnotherExtraPropertyDescription]
    i! TemplatedIntegerProperty[|Caption][|TemplatedIntegerPropertyDescription]
    s!! ReadOnlyTextProperty[|Caption][|ReadOnlyTextPropertyDescription]
    i!!|0..100 Score|Score|A templated, read-only score between 0 and 100
```
By convention, built in simple types have a single character alias (except the media types, which have short readable aliases such as img), built in complex types have a two character alias staring with t. E.G tt for task and user type aliases have a two or more character alias starting with u. E.G. t sale|us
More formally, we have:
```
t TypeName|alias
    [> description]*
    s[+][!|!!][=default][|rule]* Name[|caption][|description]
    [alias|typeName[+][!|!!][=default][|rule]* elementName[|caption][|description]]*
    [alias|typeName=(formula)[|rule]* elementName[|caption][|description]]*
```
You will notice that for most of the elements there are two MyThing properties: this is because each element is created with a name and an alias. Aliases allow you to create shortcut names to make producing a brainstorm structure document eaier.

### Descriptions
A description is written straight after the declaration, on lines starting with `>`. This is how Markdown marks a quote, and it keeps the description apart from the elements below it. A description can run over several lines, each starting with `>`, and can use full Markdown, such as bold text, lists and paragraphs. A line with only `>` separates two paragraphs. For example:
```bss
t LifeCheck|ulc
    > A regular check of my **health, weight and wealth**.
    >
    > Use this when you want to:
    > * track a few numbers each month
    > * compare them against earlier months
    s! Name|Name|What this check is called
    n Weight|Weight (kg)|My weight on the day
```
Types, extensions, choices and functions are all described this way. The description is shown as help wherever the declaration is used, in the editor and in the app. A type, extension or function without a description is flagged. A choice's description is optional.

### Templated and read-only elements
An element is marked as templated by putting `!` straight after its alias or type name, e.g. `s! Name`. Templated elements are the plan. They are copied whenever the item is copied, either on a scheduled date or when someone starts a program. All other elements are the record, and they start blank, or with their default value, in every copy.

Putting `!!` after the alias, e.g. `s!! Instructions`, marks the element as templated and read-only. It is copied like any templated element, but the element itself is locked in the copy:
* A simple element, such as text or a number, can not be changed. This is useful for text such as the description of a published lesson or plan.
* A list, such as `l:tt!!`, can not have items added or removed.
* A complex element, such as a user defined type, can not be deleted.

Only the element itself is locked. The elements inside a list item or complex element follow the rules of their own type. The original can always be edited.

See Repeating items for the full rules.

### Expected elements
Putting `+` after the alias, e.g. `n+ Weight`, marks the element as expected. It means "this should have a value", not "this must have a value":
* A simple element should have a value.
* A list should have at least one item.
* A complex element should be there.

`+` can be combined with `!` or `!!`. Write the `+` first, e.g. `s+! Name`, although `s!+ Name` is also accepted.

A missing expected value never stops a document from being imported or saved. It is flagged so it can be filled in later. See When problems are reported.

### Rules
Rules describe what a good value looks like. Each rule starts with a pipe and is written straight after the alias and any marks, before the element name. One or more spaces separate the rules from the element name, so element names can be lined up to make a type easier to read. For example, a templated, read-only score between 0 and 100:
```bss
i!!|0..100 Score|Score|A test score between 0 and 100
```
An element can have several rules, and all of them apply. For example, a user name of 3 to 20 lower case letters:
```bss
s|3..20|/^[a-z]+$/ UserName|User Name|Your login, in lower case letters only
```
Like expected elements, rules mean "should" in documents and "must" when publishing. A value that breaks a rule is flagged, but it never stops a document from being imported or saved. If the value is part of the plan, it does stop the item from being published or scheduled. See When problems are reported.

#### Ranges
A range is written as a minimum and a maximum separated by `..`. Both ends are included, so `0..100` allows both 0 and 100. Either end can be left out:

| Rule | Meaning |
|------|---------|
| `0..100` | from 0 to 100 |
| `18..` | at least 18 |
| `..100` | at most 100 |
| `100` | at most 100: a single value is a maximum |

What a range limits depends on the type of the element:
* Text: the length of the text. `s|..50` allows at most 50 characters.
* Integers and decimals: the value. `n|-10..40` allows -10 to 40.
* Dates and times: the value, written in the same format as the value. `dd|2026-01-01..2026-12-31` allows any date in 2026, and `dt|09:00..17:00` allows office hours. A date and time is written with a `T` instead of a space, such as `d|..2026-12-31T17:00`, because a space ends the rules.
* Lists: the number of items. `l:tt|1..10` allows from 1 to 10 tasks.

#### Patterns
Text can be checked against a pattern, written as a regular expression between two slashes. The slashes mark where the pattern starts and ends, so a pattern can contain pipes and spaces without being mistaken for the next rule or for the element name. A slash inside the pattern is written as `\/`.
```bss
s|/^(home|work|mobile)$/ Label|Label|Where this number reaches me
```
Most people can not read regular expressions, so when a value does not match, the flag shows the description of the element or of its type, never the pattern itself. Write the description as an example of a good value, such as "An email address, such as name@example.com".

Patterns are for experts. To let everyone else use one without reading it, give it a name by extending a simple type. See Extending types.

**Note on regular expressions:** patterns can only use these features:
* character classes such as `[a-z]`, `\d`, `\w` and `\s`
* the quantifiers `*`, `+`, `?` and `{2,5}`
* the anchors `^` and `$`
* groups `( )` and alternation `|`

Lookarounds, such as `(?=...)`, and backreferences, such as `\1`, are not supported. Leaving them out means a pattern always takes a predictable time to check, however long the value is, and that the same pattern works in other tools that read Brainstorm documents.

#### Conditions
A condition compares an element with other elements on the same item. It is written as a formula between brackets, and the value should make the condition true. For example, a project that should not end before it starts:
```bss
dd                     StartDate|Starts|When the project starts
dd|(EndDate >= StartDate) EndDate|Ends|When the project ends, on or after the day it starts
```
A condition can use everything a calculated value can (see Calculated values), plus:
* the comparisons `=`, `!=`, `<`, `<=`, `>` and `>=`
* `and` and `or`, to combine comparisons
* `not`, for the opposite of a comparison, a true or false element, or a function that gives true or false

A true or false element can be a condition on its own, and `not` turns it round. For example, a goal that you want mainly to please someone else is pointed out, until you lay it to rest:
```bss
b|(not ForSomeoneElse or Status = LaidToRest)    ForSomeoneElse|For Someone Else|True if you want this mainly to please or impress someone else
```
`not` applies to the comparison straight after it, so `not Stage = Won and Value > 100` means "the stage is not Won, and the value is over 100". `and` is worked out before `or`. Use brackets when that is not what you mean.

When a choice is compared with a name, as in `Stage = Won`, the name is read as one of the choice's values. This holds even if an element has the same name, so a total called Won can still count the deals whose stage is Won.

If an element used by a condition is blank, the condition is not checked. The blank element is flagged on its own if it is expected.

### Calculated values
A calculated value is worked out from other elements on the same item. It is declared by putting `=` straight after the alias, followed by a formula between brackets. The brackets mean the formula can contain spaces. For example:
```bss
c ProjectRiskType
    Financial
    Reputational
    HealthAndSafety
    Legal

t ProjectRisk|upr
    > The possibility that an action, event or decision leads to a bad outcome, and what it would cost.
    s+!                           Name|Name|A short name for the risk
    i|0..100                      Likelihood|Likelihood (%)|How likely the risk is to happen, from 0 to 100%
    n|0..                         Impact|Impact|What it would cost if it happened
    ProjectRiskType               RiskType|Risk Type|The kind of harm the risk would cause, which shapes what its impact means
    n=(Likelihood / 100 * Impact) RiskScore|Risk Score|The cost of the risk once its likelihood is taken into account
```
Watch the units in a formula. Likelihood is a percentage from 0 to 100, so it is divided by 100. Without that, a 50% chance of a 10,000 loss would give a risk score of 500,000 instead of 5,000.

A formula can use:
* numbers
* the names of other elements on the same item, including other calculated values
* `+`, `-`, `*` and `/`
* brackets, to control the order things are worked out in
* totals over the item's lists, such as `sum(Deals.Value)` (see Totals over lists)

The rules for calculated values are:
* A calculated value is always read-only. Users never write it in a document. If a document gives it a value, the value is ignored and flagged.
* The result has the type of the alias. `n=` gives a decimal, and `i=` rounds to the nearest whole number.
* If a value the formula uses is blank or can not be understood, or the formula divides by zero, the result is blank. It is not flagged, because the value it depends on is flagged already.
* A calculated value is never copied. It is worked out again in each copy, so `!`, `!!` and `+` are not used with it.
* Rules can follow the formula. For example, `n=(Likelihood / 100 * Impact)|..10000` flags any risk score over 10,000.
* A formula that depends on itself, directly or through other calculated values, is reported as a problem in the structure document.
* Calculated values are left out when a brainstorm is written out as a brainstorm document. A brainstorm document is a working document that only holds what people type. Reports can include calculated values.

#### Totals over lists
A formula can add up the values in the item's lists, using `sum`, `count`, `average`, `min`, `max` and `stddeviation`. Each one takes a path to the values, written as element names separated by dots, and an optional condition that picks which items count:
```bss
n=(sum(Clients.Deals.Value, Stage = Won))                              Won|Won|What you have sold so far
n=(sum(Clients.Deals.WeightedValue, Stage != Won and Stage != Lost))   Forecast|Forecast|What you can expect from open deals
i=(count(Clients.Deals, Stage = Won))                                  DealsWon|Deals Won|How many deals you have won
n=(Target - Won)                                                       Gap|Gap to Target|What is still needed to reach the target
```
The path goes down through lists. `Clients.Deals.Value` means the Value of every deal of every client. `count` counts items, so its path ends at a list rather than a value.

The rules for totals are:
* A path can only go down into the item's own lists, never up to the item it belongs to or across to other items. A formula still only sees its own item and the items inside it.
* The condition is checked against each item at the end of the path, and only the items that make it true are counted. It is written like any other condition (see Conditions).
* In a condition, a name is first looked up on the item being checked. If that item has no element with that name, it is looked up on the item the formula belongs to. This is how a condition compares each item with a total, as in the example below.
* Blank values are skipped. `average` divides by the number of values that are not blank.
* The `sum` and `count` of an empty list are 0. The `average`, `min` and `max` of an empty list are blank.

#### Finding unusual values
`stddeviation` finds the value that is a given number of standard deviations from the average of a list. Values beyond it are unusual: they may be wild cards worth a closer look, or mistakes. It takes a path, the number of standard deviations, and an optional condition, like the other totals:
```bss
n=(stddeviation(Clients.Deals.Value, 2))           UnusuallyLarge|Unusually Large|Deals worth more than this are far above the average
n=(stddeviation(Clients.Deals.Value, -2))          UnusuallySmall|Unusually Small|Deals worth less than this are far below the average
i=(count(Clients.Deals, Value > UnusuallyLarge))   UnusualDeals|Unusual Deals|How many deals are unusually large. Check them for mistakes.
```
A positive number gives a value above the average, and a negative number gives a value below it. The standard deviation is worked out from every value in the list, and a list with fewer than two values gives a blank result.

The app can use the same calculation to highlight unusual values when a list is shown as a grid.

More list calculations may be added later, as long as they stay this simple.

A formula that is used in several places can be given a name and reused. See Advanced: functions at the end of this document.

### Default values
A default value is the value an element starts with. It is declared by putting `=` straight after the alias and any marks, followed by the value without brackets:
```bss
GoalStatus=Active      Status|Status|Active, on hold, achieved, or laid to rest
i=3                    Sessions|Sessions per week|How many sessions you plan each week
s="Not started"        Stage|Stage|Where this piece of work is up to
```
Brackets are the difference: `=(...)` is a calculated value that is always worked out and can never be changed, while `=` without brackets is only a starting value. `i=0` starts at 0 and can be changed; `i=(0)` is always 0.

The rules for default values are:
* A default is filled in once, when an item is created. After that it is an ordinary value that can be changed or cleared.
* Copies are items too. A record element in a copy starts with its default instead of blank. This is how every task in a copy starts as ToDo: TaskItem declares `TaskStatus=ToDo Status`.
* A value with spaces or a pipe is written in double quotes, e.g. `s="Not started"`, because a space ends the part before the element name.
* A date can default to `today`, or to a relative date such as `+1w`. A date and time can default to `now`. These are worked out when the item is created. For a copy, today is the copy date.
* A link to a user can default to `me`, the user who creates the item. When a document is imported, that is the user who imports it. For example, `r:tu=me Author`.
* A default counts as a value, so an expected element with a default is never flagged as missing.
* An element can have a default or a formula, not both.
* A default must be one of its choice's values, and must pass the element's rules. If it does not, this is reported as a problem in the structure document.

## Built-in types
Built in types are the framework for creaing and defining the structures you want to work with. They can not be modified, but they can be extended (see below). These types provide data essential for some of the built-in services in the system such as user logins, contact details and task management. These types often need no modification to use and often are not to be used in Brainstorm documents. For example, the user that is set for a tasks owner or creator will be the user that imports the document.

### Choices
A choice is a named list of values, such as the months of the year. Choices are defined as follows:
```
c ChoiceName
    [> description]*
    [[+|-] valueName[|caption][|description]]*
```
Each value is a name, and follows the name rules. Like an element, it can have a caption and a description:
* The caption is what the app shows, for example in a drop-down list. Without one, the name is shown.
* The description is shown as help, for example when the editor offers the value.

In a brainstorm document, a value can be written as its name or its caption, in any case, so `Status: InProgress` and `Status: in progress` mean the same thing. When a brainstorm is written out as a document, the name is used. Formulas and conditions always use the name.

Choices do not have aliases, because they are written in full in documents.

A value can start with `+` or `-` to mark it as an outcome. Outcomes are only used by pipelines (see Pipelines).

To define the Months choice for example, we have the following:
```bss
c Months
    > The months of the year
    Jan
    Feb
    Mar
    Apr
    May
    Jun
    Jul
    Aug
    Sep
    Oct
    Nov
    Dec
```
Using a choice in a type is simple:
```bss
t MyType|umt
    > An example type that offers a chice of Months
    Months TheMonth|The Month|The month selected for fun
```

### Pipelines
Some items move through stages, such as a deal that goes from Lead to Won, or a job application that goes from Applied to an offer. An element that holds an item's stage is written with `p:` in front of its choice, and the app can show those items as a pipeline: a column for each stage, with the items that are at that stage.
```bss
c DealStage
    > Where a deal is up to
    Lead
    Qualified
    Proposal
    Negotiation
    + Won|Won|The client signed
    - Lost|Lost|The client said no, or went quiet
```
```bss
p:DealStage=Lead   Stage|Stage|How far the deal has got
```
The rules for pipelines are:
* The stages are the choice's values, in the order they are declared. The first value is where the pipeline starts.
* An outcome is a value that ends the pipeline: `+` marks a good outcome and `-` a bad one. The app uses them to show, for example, how many deals were won out of all the deals that finished. Outcome columns are shown last.
* `p:` can only be used with a choice that has at least one `+` outcome.
* A type can have only one pipeline element, counting the ones it gets from the type it extends.
* An element without `p:` can use the same choice as an ordinary list of values. The `+` and `-` marks then mean nothing.

### Built-in simple types
The following types are built in and the names are reserved:
* t Integer|i
    > A whole number
* t DateTime|d
    > A date and time, such as 2026-10-05T14:00 or 2026-10-05 2:00 pm
* t Time|dt
    > A time of day, such as 14:00 or 2:00 pm
* t Date|dd
    > A date, such as 2026-10-05
* t String|s
    > Text value
* t Decimal|n
    > A decmal number with a whole and fractional component expressed in decimal places
* t Boolean|b
    > A true or false value, written `true` or `false` in any case. A missing value is false.
* t Image|img
    > A picture, such as a photo on a vision board
* t Audio|aud
    > A sound recording, such as a guided meditation
* t Video|vid
    > A video, such as a lesson in a program
* t File|file
    > Any other file, such as a PDF worksheet

Dates are written year first, as in `2026-10-05`. This is the international standard (ISO 8601) used on the internet, and it means the same day in every country. The app shows dates in each user's own format.

### Media
Images, audio, video and other files are written the Markdown way. An image uses an exclamation mark, and everything else is written as a link. The text in square brackets is the caption, which is also used as alternative text for people who can not see or hear the media:
```bss
t Lesson|ul
    > One lesson in a course.
    s+!     Name|Lesson|The title of the lesson
    vid!!   Video|Video|The lesson itself
    aud!!   Meditation|Guided Meditation|A short practice to finish the lesson
    file!!  Worksheet|Worksheet|A worksheet to fill in
    l:img   Pictures|Pictures|Pictures that go with the lesson
```
```bsd
Lesson: Facing your fears
    Video: [Measuring the monster](lessons/measuring-the-monster.mp4)
    Meditation: [Calm before action](audio/calm-before-action.mp3)
    Worksheet: [Fear inventory](worksheets/fear-inventory.pdf)
    Pictures:
    * ![The monster, revealed as a small furry ball](images/monster.jpg)
```
A plain web address or file path is also accepted.

The file can be one that has been uploaded to Brainstorm, or a link to a file elsewhere. In a published program, media must be uploaded to Brainstorm: links to files elsewhere can break, and paid content must only be shown to the people who bought it. A link to a file elsewhere in a program's plan is flagged when the program is published. See When problems are reported.

### Built-in complex types
* t List<T>|l:x
    > A generic list of objects where x is the name or alias of the objects that the list holds
* t Reference<T>|r:x
    > A link to an item of type x that lives somewhere else in the brainstorm, such as a ticket that another ticket depends on. A list of links is written `l:r:x`. See Links.
* t Pipeline<T>|p:x
    > The stage of an item that moves through the stages of the choice x. See Pipelines.
* t TaskItem|tt
    > A task object that we can use in kanban boards
**Definition**
```bss
t TaskItem|tt
    > A TaskItem is anything that needs to have action taken to implement some desired outcome. This can be used in a todo list, a list of things that must be done to get controls in place for a risk, a projects kanban board, etc.    
    s!!+ Name|Task Name|A short name to identify the task without a full description
    s!+ Description|Description|A full description of what the task entails, what needs to be done and any other information pertinent to the execution and completion of the task
    d CreatedDate|Created Date|The date and time the task was created
    TaskStatus=ToDo Status|Status|The status of the task
    i Priority|Priority|When a task appears in a list priority can be used to sort them.  
    d! StartDate|Start Date|When the task will be started
    d! DueDate|Due Date|The date and time the task should be done by
    n!|0.. Estimate|Estimate (hours)|How long the task is expected to take
    n!|0.. Cost|Cost|What the task costs, if anything
    r:tc AssignedTo|Assigned To|Who will do the task
    r:tu=me CreatedBy|Created By|The user that created the task
    d CompletedOn|Completed On|The date and time the task was done
    l:tt SubTasks|Sub-tasks|The task has been broken down into smaller tasks and can be considered complete when the sub tasks are all done.
    ts Schedule|Repeats|Optional. Makes the task repeat, such as a weekly gym session or a monthly bill payment.
    l:tcm Comments|Comments|Optional. Your thoughts about the task, kept apart from its description
```
StartDate and DueDate are templated so that relative dates, such as `+2d`, are carried into copies. Any task with a Schedule repeats, wherever it is. See Repeating items.
* t Contact|tc
    > A contact with contact details such as a phone number, email address, etc. 
    **Definition**
```bss
t contact|tc
    > An contact is a party that has contact details
    s Name|Name|The first name of the contact 
    s Surname|Surname|The surname of the contact
    s CompanyName|The name of the company if the contact is a contact person in a company
    l:tcm Comments|Comments|Optional. Notes about the contact
```
* t ContactDetail|tcd
    > A single detaial about a contact, for example, the phone number or email address.
    ContactDetailType Type|Contact Detail Type|One of the predefined contact types in the ContactDetailType choice.
    s Name|Label|An optional note such as "home" or "after hours"
    s Value|Value|The actual contact detail value
    b IsPrimary|Is Primary|The one to use first when there is more than one of this type.
* t User|tu
    > Another user of the system which also has contact details.
    **Definition**
```bss
t User|tu
    > A user of the system
    s Name|Login|The user login name (usually an email address)
    tc Contact|Contact|The person or party that the user represents    
    dd CreatedOn|Created On|The date the user record was created
    b IsActive|Is Active|True if the user is actively using the system.
```
* t Goal|tg
    > Something you want to achieve, why you want it, and what stands in the way.
    **Definition**
```bss
t Goal|tg
    > Something you want to achieve, why you want it, and what stands in the way.
    s+!      Name|Goal|A short name for the goal
    s!       Description|Description|What achieving this goal looks like
    dd!      TargetDate|Target Date|When you want to have achieved it
    i!|1..   Priority|Priority|1 is the most important
    l:s      Reasons|Why|Why you want this. Keep asking why until you reach the real reason.
    l:tm     Milestones|Milestones|Measurable points on the way to the goal
    l:tt     Actions|Actions|The tasks that move you towards the goal. A task with a schedule is a regular action.
    l:tr     Risks|Risks|What could stop you
    l:tg     SubGoals|Sub-goals|Smaller goals that make up this one
    l:tcm    Comments|Comments|Optional. Your thoughts about the goal, kept apart from its description
```
* t Milestone|tm
    > A measurable point on the way to a goal.
    **Definition**
```bss
t Milestone|tm
    > A measurable point on the way to a goal.
    s+!      Name|Milestone|What will be true when you reach it
    dd!      TargetDate|Target Date|When you want to reach it
    s!       Measure|Measure|What you measure, e.g. body fat (%)
    n!       Target|Target|The value you want to reach
    n        Actual|Actual|The value you reached
    l:tcm    Comments|Comments|Optional. Notes about the milestone, such as how you reached it
```
* t Risk|tr
    > Something that could stop you or hurt you. A risk is rated twice: as it is now, and as it will be once its controls are in place. Controls often overlap, so the remaining risk is rated by judgement rather than worked out from the controls.
    **Definition**
```bss
t Risk|tr
    > Something that could stop you or hurt you: how likely it is, how bad it would be, and what is left once it is controlled.
    s+!                                                   Name|Risk|e.g. "Losing income if I get sick"
    s!                                                    Cause|Cause|Why it might happen
    s!                                                    Consequence|Consequence|What would happen if it did
    RiskType+!                                            RiskType|Risk Type|The main kind of harm it would cause
    i+|1..10                                              Likelihood|Likelihood|1 is very unlikely, 10 is almost certain
    i+|1..10                                              Severity|Severity|How bad it would feel, from 1 (meh) to 10 (life changing)
    n|0..                                                 FinancialImpact|Financial Impact|What it would cost in money, if it has a cost
    n=(Severity * Likelihood / 10)                        RiskScore|Risk Score|Ranks every risk on the same scale
    n=(FinancialImpact * Likelihood / 10)                 ExpectedLoss|Expected Loss|The cost once its likelihood is taken into account
    b                                                     Accepted|Accepted|True if you can live with this risk and will not control it
    l:trc                                                 Controls|Controls|What you will do about it
    i|1..10|(ResidualLikelihood <= Likelihood)            ResidualLikelihood|Likelihood After Controls|How likely it is once your controls are in place
    i|1..10|(ResidualSeverity <= Severity)                ResidualSeverity|Severity After Controls|How bad it would be once your controls are in place
    n|0..|(ResidualFinancialImpact <= FinancialImpact)    ResidualFinancialImpact|Financial Impact After Controls|What it would cost once your controls are in place
    n=(ResidualSeverity * ResidualLikelihood / 10)        ResidualScore|Residual Score|The risk that is left once your controls are in place
    n=(ResidualFinancialImpact * ResidualLikelihood / 10) ResidualExpectedLoss|Residual Expected Loss|The cost that is left once your controls are in place
    l:tcm                                                 Comments|Comments|Optional. Your thoughts about the risk
```
Every risk has a Severity, including financial ones, so that all risks can be ranked together by RiskScore. A financial risk also has a FinancialImpact in money, which gives its ExpectedLoss. Each field has one unit: Severity is always 1 to 10, and FinancialImpact is always money.
* t Control|trc
    > Something you do to stop a risk happening, or to limit the damage if it does.
    **Definition**
```bss
t Control|trc
    > Something you do to stop a risk happening, or to limit the damage if it does.
    s+!             Name|Control|e.g. "Take out income protection insurance"
    ControlType+!   ControlType|Type|Prevent stops the risk happening. ReduceImpact limits the damage if it does.
    i!|0..100       Effectiveness|Effectiveness (%)|How much of the risk you expect this control to remove
    n!|0..          Cost|Cost|What it costs to put the control in place
    l:tt            Tasks|Tasks|The tasks that put the control in place, in the order they are done
    l:tcm           Comments|Comments|Optional. Notes about the control, such as whether it is working
```
Effectiveness and Cost are used together to decide whether a control is worth doing. A Prevent control lowers the residual likelihood of its risk, and a ReduceImpact control lowers its residual severity or financial impact.
* t Assessment|ta
    > A snapshot of where you are now, so that you can compare it later.
    **Definition**
```bss
t Assessment|ta
    > A snapshot of where you are now, so that you can compare it later.
    s+!             Name|Assessment|e.g. "Health check"
    dd=today        Date|Date|When the measurements were taken
    ts              Schedule|Repeats|How often to take the snapshot
    l:tai+          Items|Measurements|What you measure
    l:tcm           Comments|Comments|Optional. Notes about this snapshot
```
* t AssessedItem|tai
    > One thing you measure in an assessment.
    **Definition**
```bss
t AssessedItem|tai
    > One thing you measure in an assessment.
    s+!!            Name|Measure|e.g. Weight. Locked in copies so that results stay comparable.
    s!              Area|Area|The part of life it belongs to, e.g. Health
    s!              Unit|Unit|e.g. kg, $, or 1-10 for a rating
    s!              HowMeasured|How Measured|How to measure it the same way each time
    n+              Value|Value|The measurement
    l:tcm           Comments|Comments|Optional. Notes about the measurement, such as anything unusual on the day
```
* t Comment|tcm
    > A comment on an item: who said what, and when. Comments let people add their thoughts to an item without changing its description, and they are never required. The built-in item types, such as TaskItem, Goal, Risk and Contact, already have a list of comments. To give your own types comments, add `l:tcm Comments`.
    **Definition**
```bss
t Comment|tcm
    > A comment on an item: who said what, and when.
    s+        Name|Comment|What you want to say
    d=now     Date|Date|When the comment was made
    r:tu=me   Author|Author|Who made the comment
```
Because the date and the author are filled in for you, most comments are a single line of text. A comment's text is its Name, so a plain bullet is a whole comment:
```bsd
    Comments:
    * Check the layout on small phones
    * Agreed, I will test it on my old phone|2026-10-02T09:30|[[Sam Jones]]
```
* t Schedule|ts
    > A schedule says when a new copy of an item should be created. See Schedules for how to write one.
    **Definition**
```bss
t Schedule|ts
    > A schedule says when a new copy of an item should be created.
    s Name|Name|A short name for the schedule, written as something to do, e.g. "Monthly check-in". A copy that is not a task is listed in the user's tasks as "Do" and this name.
    Frequency Repeats|Repeats|How often a new copy is created
    i|1.. Every|Every|Repeat every N periods: 1 is every month, 2 is every other month. Missing means 1.
    dd StartDate|Starts|The date of the first copy. It also sets the day, e.g. starting on the 5th means monthly on the 5th. It can be a relative date, such as +1d.
    dd EndDate|Ends|Optional. No copies are created after this date.
    dt DueTime|Due At|Optional time of day the copy is due, in the user's own time zone
    s On|On|Optional. Which days, e.g. "Mon, Wed, Fri", "weekdays", or "last Fri" for a monthly schedule
    i|1.. Times|Times|Optional. Stop after this many copies, e.g. 12 sessions
    s Rule|Calendar Rule|For experts: an iCalendar RRULE, used instead of Repeats, Every, On, Times and EndDate
    dd NextDate|Next Date|Set by the system: the date the next copy will be created
```

### Built-in choices
ContactDetailType - used in contact detail to select a type. 
**definition**
```bss
c ContactDetailType
    > The kind of contact detail, such as an email address or a phone number
    Email
    PhoneNumber|Phone number
    WorkPhone|Work phone
    MobilePhone|Mobile phone
    SocialMediaUrl|Social media
    Address
    Other
```
Frequency is the choice for setting up schedules
```bss
c Frequency
    > How often a schedule repeats
    Daily
    Weekly
    Monthly
    Quarterly
    Yearly
```
TaskStatus is the set of statuses that a task can be in: used for the kanban board and tracking projects
```bss
c TaskStatus
    > Where a task is up to
    ToDo|To do|Not started yet
    InProgress|In progress|Someone is working on it
    InReview|In review|Finished, and waiting to be checked
    Blocked|Blocked|Can not go on until something else happens
    OnHold|On hold|Paused on purpose
    + Done|Done|Finished
    - Dropped|Dropped|Will not be done
```
RiskType is the main kind of harm a risk would cause
```bss
c RiskType
    > The main kind of harm a risk would cause
    Social
    Financial
    Emotional
    Legal
    Health
    Reputational
```
ControlType says how a control deals with a risk
```bss
c ControlType
    > How a control deals with a risk
    Prevent|Prevent|Stops the risk from happening
    ReduceImpact|Reduce impact|Limits the damage if the risk does happen
```

### Extending types
It can often be the case that you want some new peice of data on an existing type. For example, in the Task you may want to add a new element to allow you to delegate that task to a contact. In this case it is better, in fact essential, to extend the task rather than copy and paste it's definition, so that the system can still recognise the item as a task. Here is how you would implement such a change:
```bss
x DelegatedTask:TaskITem|udt
    > A task that can be delegated to a contact: useful for team management and sharing tasks.
    tc DelegatedTo|Delegate|The person that is responsible for the task completion.
```
The declaration of this is similar to the declaration of a normal type, but starts with x and must have a colon between the name of the new type and the type that it extends. The description of this new type will be used instead of the description of the type that it extends. All other elements can be appended in the same manner as they are declared in a normal complex type. Any duplicates are simply ignored.

An extended type can be used anywhere the type it extends can be used. For example, a list of tasks can hold delegated tasks. See Shorthand for how a list item's type is chosen.

An extended type can also say that a list it got from the original type holds its own kind of item. This is the one case where a repeated element is not ignored. For example, a ticket's sub-tasks should be tickets too, with their own comments and links:
```bss
x Ticket:TaskItem|uti
    > A piece of work in a project. A big ticket can be broken down into sub-tickets.
    l:uti   SubTasks|Sub-tickets|Smaller tickets that make up this one
```
The list can only be changed to hold an extension of the type it held before. A list of tickets is still a list of tasks, so everything that works with tasks, such as the kanban board, still works.

Simple types can be extended too. This is how a rule is given a name, so that people can use it without having to read or write it. The rules are written after the alias, each starting with a pipe. For example, an email address is text that matches a pattern:
```bss
x EmailAddress:s|em|/^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$/
    > An email address, such as name@example.com
```
Anyone can now use `em` like any other type, without knowing the pattern:
```bss
em+ Email|Email Address|Where to send reminders
```
An extended simple type only adds rules and a description. It has no elements. If an element that uses it adds rules of its own, both sets of rules apply.

### Name rules
For a name to be valid, these simple rules must be followed:
* Names must start with a letter or underscore (_).
* Names can contain alpha-numeric characters and the underscore (_).
* Names are not case sensitive, so `Goal`, `goal` and `GOAL` are the same name. This applies to the names of types, aliases, elements, choices and functions.

## Brainstorm documents
A brainstorm document is a text document, similar to markdown, that allows us to quickly list items in meaningful ways, without having to do data capture on some form or spreadsheet. This saves us from having to navigate between controls on pages, click save a hundred times and wait for excel to think when we want to add an item.
Some use cases are quickly creating action plans, identifying risks and controls, setting goals and planning how to achieve them. For example, using the built-in Goal type (see Built-in complex types), I can create a goal with a list of tasks easily as follows:
```bsd
Goal: Book a holiday
    Actions:
    * Find a sunny destination wich fits into my budget
    * find the best flights
    * book a hotel
    * get travel insurance
    * hire a car or book transfers to the destination from the airport
    * look into local acivities and excursion
```

### Shorthand
Any item can be written on a single line by listing its values in the order its elements are declared, separated by pipes. Anything not on that line is written on indented lines below it.

The rules are:
* A name and its value are always separated by a colon. Spaces around the colon are ignored, so `Goal:Book a holiday` and `Goal: Book a holiday` mean the same thing.
* A line that is not indented starts a new item. The word before the colon is a type.
* An indented line sets an element of the item above it. The word before the colon is an element.
* Only the first colon on a line separates the name from the value, so values can contain colons, as in `DueTime: 9:00 am`.
* A list element is written as its name and a colon with no value, followed by its items, each starting with `*`.
* A list item has the type of its list, unless it uses an element that only an extension of that type has. Then it becomes that extension. For example, in a list of tasks, an item with a `DelegatedTo:` line becomes a DelegatedTask (see Extending types).
* To choose the type of a list item yourself, start the item with the type and a colon, e.g. `* udt: Book the venue`. A word that is not a type the list can hold is just part of the item's text, so `* Achieved: I reached 3rd dan` is a comment that starts with "Achieved". This makes the item a DelegatedTask even before you know who to delegate it to. It is also how to choose when two extensions share an element name and the system can not tell which one is meant. Until you choose, such an item is flagged.
* Wherever a document names a type, its alias can be used instead. `udt: Book the venue` and `DelegatedTask: Book the venue` mean the same thing.
* A complex element, such as a Schedule, can use shorthand after its colon. Its other elements are indented one level further below it.
* Values on the line follow the order the elements are declared in the type, starting at Name. The type description belongs to the type, not to each item, so it never takes a position.
* Only simple elements take a position: text, numbers, dates, times, true/false values, choices and single links. Lists and complex elements are skipped when counting positions and are always written below the line. Calculated values are skipped too, because they are never written in a document.
* Adding a new element at the end of a type keeps the positions of the elements before it, so documents written before the change still work. Adding one in the middle moves the positions of everything after it.
* Leave a position empty to skip it. For example, `a||c` skips the second value.
* Any element can be written on its own indented line below, as the element name, a colon, then the value.
* Values on the shorthand line can not contain a pipe. Write such a value on its own line below instead.

Because an extended type adds its elements after those of the type it extends, shorthand written for the original type still works for the extended one.

Using the Goal type, the values on the line are Name, Description, TargetDate and Priority, in that order. Here the Priority is written below the line instead, and the reasons are a list:
```bsd
Goal: Book a holiday|Sun, sea and no laptop|2027-03-01
    Priority: 2
    Reasons:
    * I haven't had a proper break in two years
    Actions:
    * Find a sunny destination wich fits into my budget
    * find the best flights
    * book a hotel
```

### Relative dates
A date can be written relative to a start date, using `+` followed by a number and a unit: `d` for days, `w` for weeks, `m` for months and `y` for years. Words are also accepted, so `+2w` and `+2 weeks` mean the same thing.

Relative dates are how a plan says "when" without knowing when it will be followed. They stay relative in the document they are written in, and the app shows them as, for example, "2 weeks after start". They are given a real date when a copy is made. See Repeating items.
```bsd
Goal: Run a 5k|Go from the sofa to running 5k without stopping|+12w
    Milestones:
    * Run 1k without stopping|+3w
    * Run 3k without stopping|+8w
    Actions:
    * Buy running shoes
        DueDate: +2d
```

### Links
A link points to an item that lives somewhere else in the same brainstorm, such as a ticket that another ticket depends on. It is written as the item's name between double square brackets, as in many wikis and note-taking apps:
```bsd
* Build the login page
    DependsOn:
    * [[Set up the database]]
    * [[Design the login screen]]
```
If two items have the same name, the link is flagged. Add the names of the items above it, separated by slashes, to say which one you mean: `[[Website/Set up the database]]`.

The rules for links are:
* Links only reach items in the same brainstorm, with one exception: links to users and contacts reach the users and contacts in the system that the importing user has access to. For now, users and contacts are linked by name, e.g. `[[Sam Jones]]`. A contact's name for a link is its full name: its Name followed by its Surname. What happens when one can not be found depends on the import mode (see Importing and exporting).
* When a document is imported, each link is turned into the internal id of the item it names. From then on, renaming an item does not break the links to it. When the brainstorm is written out as a document again, each link is written with the item's current name.
* A link to an item that does not exist, or to a name that is not unique, is flagged.
* A link never owns the item it points to. Deleting an item does not delete the items that link to it: their links are flagged instead.
* Links that go round in a circle, such as two tickets that each depend on the other, are flagged.
* An item name containing `/`, `[` or `]` is pointed out, because a link can not reach it. `/` separates the names in a link, and brackets end one.
* Tags are not part of a name when it is linked to (see Tags).
* When an item is copied, a link to another item inside the copy points to that item's copy. A link to an item outside the copy still points to the original.

Renaming an item in a text editor does not update the links to it, because the document only holds names. Those links are flagged when the document is imported, until they are changed to the new name.

### Tags
A tag is a `#` followed by a name, such as `#budget`. Tags can be written in any text: names, descriptions, other text values and notes. They make items easy to find:
```bsd
Goal: Book a holiday #family
    Actions:
    * Find a sunny destination #budget
    * Book the flights #budget #urgent
```
The rules for tags are:
* A tag starts at the beginning of the text or after a space, and follows the name rules. So `C#`, `Room #4` and a heading such as `# Plans` are not tags.
* Tags are not case sensitive, so `#Budget` and `#budget` are the same tag.
* A tag belongs to the item whose text it is in. The items inside it do not get its tags.
* Tags stay in the text as they are written, and the app shows them as labels. A link leaves them out, so `[[Book the flights]]` finds the last item above.

In the app, tags are used to filter the tree, the kanban board and My tasks.

### When problems are reported
In the default import mode, import never fails (see Importing and exporting for the stricter modes). A value the system can not understand, such as `Weight: about 80`, an expected value that is missing, or a value that breaks a rule, is kept as written and flagged so it can be fixed later in the document or in structured mode. Nothing written is ever lost:
* An element the item's type does not have, such as `Mood: happy` on a task, is kept with anything indented below it, and flagged. If a new version of a structure renames an element, the old value is still there to move across.
* A line inside an item that is neither an element nor a list item is kept as a note, and flagged.

While writing, flags are quiet hints. They are pointed out again when an item is marked as done, but nothing is blocked.

Publishing and scheduling are stricter, because other people and future copies depend on the plan being complete. A program can not be published while its plan has problems, and a schedule does not create a copy while the original's plan has problems. The author is told what needs fixing.

The plan is everything that gets copied: templated elements, lists, and the templated elements of each item in those lists. Record elements, such as Weight in a LifeCheck, are not checked, because they start blank, or with their default value, in every copy anyway.

# The Brainstorm application
The application supports editing brainstorm structure documents and uses these documents to generate a UI based on the defined types in the brainstorm structure. A user has a number of brainstorms available which can be viewed as a document/text, or as structured objects (in structured mode) where the UI is based on the structure document that the brainstorm document uses. The structure document is not used to generate code, but instead to create an in-memory object model that is tolerant of user errors. 
When a user is viewing their brainstorm in structured mode the layout is driven by the model defined in the brainstorm structure document.
Out of the box, all strucuters inherit tasks, goals, risks, controls, contacts, contact details, schedule, assesment and assessed items which are pre-defined. Goals, risks, controls and assessments are included in the structure document when a new brainstorm structure document is created, allowing users to remove these items. Tasks, contacts, contact details and users are not deletable. This is because tasks are the most meaningful thing in a brainstorm and are a first class structure. Tasks must be supported by a kanbas board in the UI.
The system allows users to keep a catalog of Brainstorm structure documents (on disk or in a database) and a catalog of brainstorms. Some brainstorms, like assessment-style brainstorms must support repeating so that the assessed items can be measured and compared against historic values. For example, give an assessment style structure as follows:
```bss
t LifeCheck|ulc
    > A regular check of my health, weight and wealth
    s+! Name|Name|What this check is called
    n! TargetWeight|Target Weight (kg)|The weight I am aiming for
    ts Schedule|Schedule|How often I do this check
    n+ Weight|Weight (kg)|My weight on the day
    i RestingHeartRate|Resting Heart Rate|Beats per minute first thing in the morning
    n NetWorth|Net Worth|Assets minus debts
    l:tt+ Steps|Steps|The things I do to complete the check
```

### Importing and exporting
A brainstorm is set up from a document, and then managed in the app. It can be written out as a document at any time, edited in any text editor, and imported again. A document is imported in one of two ways:
* **As a new brainstorm.** Everything in the document is added.
* **Into an existing brainstorm**, to update it. The items in the document are matched with the items already there, so that their history is kept (see History).

Items are matched by name. An item matches the item of the same type with the same name in the same place: under the matching parent, and in the same list. Names are compared ignoring case and tags. A matched item takes the values in the document, an item that is only in the document is added, and an item that is only in the brainstorm is deleted.

So renaming an item in a document means "delete this one and add that one", and the old item's history would go with it. Before an import goes ahead, the app lists what will be added and what will be deleted, and the user can pair a deleted item with an added one to say it was renamed. Two items with the same name in the same place can not be matched by name, so the user pairs them in the same way.

A new copy of a templated item is added rather than matched. A new item named after an original in the same list, followed by a date, as copies are named (see Occurrences and starts), is imported as a new copy of it: its plan comes from the original, and its other values from the document. For example, `* Where I am 2026-12-04`, written in the same list as `Where I am`, adds this month's assessment. Copies already in the brainstorm are matched by their dated names, like any other item.

Editing a brainstorm as text in the app works the same way: saving the text imports it into the brainstorm.

Calculated values are not in the document, but they are worked out again on import. Links only reach items in the same brainstorm, so importing into one brainstorm never breaks the links in another.

#### Import modes
How carefully an import is checked is up to the user. Each user chooses a default mode in their settings, and can choose a different mode for any single import.

| Mode | What happens when something does not check out | Suits |
|------|-----------------------------------------------|-------|
| **Default** | The import goes ahead, and problems are flagged. A link to a contact that does not exist creates the contact. | Individuals capturing their thoughts |
| **Referential** | The import is refused if any link can not be found: to an item in the document, or to a user or contact in the system. Contacts written in the document count, and are added to the user's contacts. | Teams and shared projects |
| **Strict** | The import is refused if anything is wrong: links, rules, expected values or conditions. This is the same check as publishing, applied to the whole document. | Authors, organisations and data from other systems |

In every mode:
* **An import never creates a user.** A user is someone who can log in, so a line of text must never create one. In the default mode, a link to an unknown user is kept as written and flagged.
* **A refused import changes nothing.** There are no half-imported brainstorms.
* **A refused import lists every problem at once,** with the line it is on, so that everything can be fixed before the next try.

Later, an organisation may be able to require a minimum mode for its members, for example that everyone imports in referential mode.

#### Inviting people
When a document links to a user by email address, such as `[[sam@example.com]]`, and nobody with that address uses Brainstorm, the app can invite them. Invitations are never sent automatically: the app lists the unknown addresses, and the importing user chooses who to invite. Each person invited gets one email.

Until they join, the link points to a pending user. A pending user counts as found, so the import is not refused in the referential or strict modes. When the person joins, everything already linked to them, such as the tasks assigned to them, is waiting in their account.

### History
Some things change over time, and some are taken again and again. The app treats them differently:
* **Things that change**, such as goals, tasks, deals and job applications, keep a history: each change to each element, with the old and new values, when it was made and who made it, and when the item was added or deleted. This shows, for example, how long deals stay at each stage, or how many job applications turn into interviews.
* **Copies made from a template**, such as each month's assessment or each evening's review, are snapshots. Each one is the truth at the moment it was taken, and the next copy shows what changed since, so the copies themselves are the history. Weight over a year is the weight in each month's copy. Their changes are not tracked, and nothing is locked: people are trusted to be honest with themselves.

A program someone has started is their own plan to manage, so it keeps a history like anything else that changes. Only the copies made by schedules, or by hand, are snapshots.

History is kept by the app, not written in documents. A document is a snapshot of how things are now. History belongs to the brainstorm: it is deleted with the brainstorm, or earlier if the user sets a retention policy.

### Repeating items
Structures are a way of knowing what tasks need to be done and why. Repeating an item is how the system creates the next round of tasks.

An item with a schedule repeats wherever it is:
* An item in a list gets its copies added to the same list. For example, a task with a schedule in the actions of a goal adds a new task to those actions each time it repeats.
* A root item, which is an item that is not inside another item, gets its copies added as new root items.
* A single complex element that is not in a list can not hold a second copy, so a schedule there is ignored.

Lists of copies grow over time. The copies are the record of what was done, so they are kept. The app hides finished tasks by default, and users can set a retention policy to delete old data after a period of their choice.

#### Occurrences and starts
The item that is copied is the original. There are two kinds of copy:
* An **occurrence** is one more round of a repeating item. It is created by a schedule, or by hand.
* A **start** is someone's own copy of a whole program, created when they start following it.

Both kinds of copy follow the same rules:
* Only templated elements, marked with `!` or `!!`, are copied. All other elements start blank, or with their default value, so they can be filled in on the day and compared with earlier copies.
* Elements marked with `!!` are locked in the copy: simple values can not be changed, lists can not have items added or removed, and complex elements can not be deleted.
* Lists are always copied, whether or not they are marked. Each item in a list is created using the `!` and `!!` marks of its own type. A list of simple values, such as `l:s`, is copied as it is.
* A templated complex element is copied the same way: its own elements follow the `!` and `!!` marks of its type.
* If Name is templated, an occurrence gets the date added to it, e.g. "Life check 2026-10-01".
* Record elements start with their default value, or blank if they have none. Because TaskItem's Status defaults to ToDo, every task in the copy, at any depth, starts as ToDo.

Dates in the copy are worked out from the copy date. For an occurrence created by a schedule, this is the scheduled date. For an occurrence created by hand, it is the date the user picks. For a start, it is the day the follower starts the program.
* A relative date becomes the copy date plus its offset. A task due `+2d` in a copy made on 2026-10-01 is due on 2026-10-03.
* A task with no DueDate is due on the copy date, at the DueTime of the schedule if one is set.
* A date that is not relative is copied as it is.

The one difference between the two kinds of copy is what happens to the schedules inside them. Schedules are never templated: the kind of copy decides.
* An occurrence has no schedules. This stops copies from creating more copies, so only the original repeats. An occurrence of a repeating task is an ordinary task.
* A start keeps its schedules, so the habits in a program begin repeating on the day the follower starts it. A relative StartDate, such as `+1d`, becomes a real date.

No occurrence is created while the original's plan has problems. See When problems are reported. A scheduled date that passes while the plan has problems is skipped. It is not caught up automatically once the plan is fixed.

A user can also create an occurrence by hand, for example to make up for a skipped date. The user picks the date of the copy, which defaults to today. The plan must have no problems, just as for a scheduled occurrence. Creating an occurrence by hand does not change the schedule.

A scheduled occurrence of an item that is not a task, such as a monthly assessment, still needs doing. So it appears in the user's tasks as "Do" followed by the name of its schedule, such as "Do Monthly check", with a link to the occurrence in the web app, where it is filled in. It leaves the user's tasks once none of its expected values are missing, or when the user ticks it off.

To follow your own program, you start it like anyone else.

For example, using the LifeCheck type above, this is the original a user writes:
```bsd
LifeCheck: Life check
    TargetWeight: 78
    Schedule: Monthly check|Monthly||2026-10-01
    Weight: 82.5
    Steps:
    * Weigh myself before breakfast
    * Take my resting heart rate
    * Update the net worth spreadsheet
```
And this is the copy the system creates on 2026-10-01. Weight is blank, and each step is a new task with the Status ToDo and a DueDate of 2026-10-01:
```bsd
LifeCheck: Life check 2026-10-01
    TargetWeight: 78
    Steps:
    * Weigh myself before breakfast
    * Take my resting heart rate
    * Update the net worth spreadsheet
```
Any task repeats once it has a schedule. This one is created every other Wednesday:
```bsd
TaskItem: Visit mom|Pop round for tea and help with the garden
    Schedule: Every other Wednesday|Weekly|2|2026-09-30
        DueTime: 9:00 am
```
A repeating task can also sit inside a goal, so that a regular action stays with the goal it serves. Each session is added to the goal's actions, and appears on the goal's kanban board and in the user's tasks:
```bsd
Goal: Get fit by summer|Strong, lean and able to run 10k|2027-06-01
    Actions:
    * Gym session|Strength and cardio, 45 minutes
        Schedule: Weekly gym|Weekly||2026-10-05
    * Book a session with a personal trainer
```

### Schedules
A schedule is written as a Schedule element under the item that repeats. The line after `Schedule:` is shorthand for its Name, Repeats, Every and StartDate, and the other elements go on the lines below it:
```bsd
* Gym session|Strength and cardio, 45 minutes
    Schedule: Gym|Weekly||2026-10-05
        On: Mon, Wed, Fri
        DueTime: 7:00 am
```
The app always shows a schedule back in plain words, with the next few dates, so you can check it says what you meant. For example: "Every Mon, Wed and Fri at 7:00 am, starting 5 Oct 2026. Next: 5 Oct, 7 Oct, 9 Oct."

#### Recipes
| What you want | What you write |
|---------------|----------------|
| Every day | `Daily` |
| Every weekday | `Weekly`, with `On: weekdays` |
| Three times a week | `Weekly`, with `On: Mon, Wed, Fri` |
| Every other Tuesday | `Weekly`, Every `2`, with `On: Tue` |
| On the 1st of every month | `Monthly`, starting on the 1st of a month |
| The last Friday of every month | `Monthly`, with `On: last Fri` |
| Every three months | `Quarterly` |
| Once a year | `Yearly`, starting on the date you want each year |
| 12 weekly sessions | `Weekly`, with `Times: 12` |
| Daily for 30 days, starting the day after a program starts | `Daily`, starting `+1d`, with `Times: 30` |
| Just once | No schedule: give the task a DueDate instead |

#### Writing On
`On` says which days a schedule falls on. It is a list of days separated by commas:
* Day names: `Mon`, `Tue`, `Wed`, `Thu`, `Fri`, `Sat` and `Sun`. Full names, such as `Monday`, are also accepted.
* `weekdays` means Monday to Friday, and `weekends` means Saturday and Sunday.
* For a monthly or yearly schedule, a day can have a position in front of it: `1st`, `2nd`, `3rd`, `4th` or `last`. For example, `1st Mon` or `last Fri`.

If `On` can not be understood, it is flagged, like any other value that can not be understood.

#### Times and time zones
DueTime is always the user's own local time. A program that says 7:00 am means 7:00 am wherever the person following it lives, which matters because programs can be followed anywhere in the world.

#### Calendar rules, for experts
Schedules are worked out using the recurrence rules of iCalendar, the standard used by Google Calendar, Outlook and Apple Calendar. This means the app can also publish your tasks and habits as a calendar feed, so that they appear in the calendar you already use.

Anything the elements above can not express can be written as an iCalendar rule in the Rule element. A Rule is used instead of Repeats, Every, On, Times and EndDate. StartDate and DueTime still apply. For example, the last working day of every month:
```bsd
    Schedule: Month end report|||2026-10-01
        Rule: FREQ=MONTHLY;BYDAY=MO,TU,WE,TH,FR;BYSETPOS=-1
```
The full rule syntax is defined in section 3.3.10 of the iCalendar standard, RFC 5545. Like patterns, rules are for experts: most schedules never need one.

For those building the system, each Schedule is turned into an iCalendar rule:

| Schedule element | iCalendar rule part |
|------------------|---------------------|
| Repeats | `FREQ`. Quarterly is `FREQ=MONTHLY` with `INTERVAL=3`. |
| Every | `INTERVAL` |
| On | `BYDAY`, with positions such as `1MO` or `-1FR` |
| Times | `COUNT` |
| EndDate | `UNTIL` |
| StartDate and DueTime | `DTSTART`, as a floating local time |

# Advanced: functions
This section is for authors of structure documents who find themselves writing the same formula more than once. Nobody needs functions to write a brainstorm document, and most structures will never need them.

A function is a formula with a name. It lets a formula be written once and used in any calculated value or condition. A function that returns true or false is sometimes called a predicate, and it is used in conditions. There is only one kind of declaration, and the result type tells you which kind of function it is.

### Declaring a function
A function is declared with a line starting with f, followed by the alias of the type it returns, the name of the function and its parameters in brackets. The lines below it that start with `>` are its description, and the last line is the formula, starting with `=`:
```
f resultAlias FunctionName([alias parameterName[, alias parameterName]*])
    > description
    = formula
```
For example, a calculation and a predicate:
```bss
f n PercentOf(n Percent, n Amount)
    > A percentage of an amount, e.g. 50% of 10,000 is 5,000
    = Percent / 100 * Amount

f b OnOrAfter(dd Later, dd Earlier)
    > True when the later date is the same as or after the earlier one
    = Later >= Earlier
```
A function is used by writing its name followed by the values to give it, in brackets:
```bss
n=(PercentOf(Likelihood, Impact))   RiskScore|Risk Score|The cost of the risk once its likelihood is taken into account
dd|(OnOrAfter(EndDate, StartDate))  EndDate|Ends|When the project ends which must be on or after the day it starts
```
The description is required. A function without one can not be understood by the next person who reads it, and it is shown as help wherever the function is used. A description can run over several lines, each starting with `>`, just like a type description.

### The rules for functions
* A function is a single formula. It has no statements, variables or loops. The formula can use everything a calculated value or condition can.
* A function can only use its own parameters. It can not see the elements of the item it is used on, so everything it needs must be passed in.
* A function can call other functions, but it can not depend on itself, directly or through other functions. This is reported as a problem in the structure document. It means every formula always finishes.
* Functions are not values. A function can not be passed to another function or stored in an element.
* Parameters use the aliases of the built-in simple types, such as `n`, `i`, `dd` and `b`. Giving a function a value of the wrong type is reported as a problem in the structure document.
* If a value given to a function is blank or can not be understood, the result is blank. A condition with a blank result is not checked.
* Functions are declared in a structure document, in the same way as types. Function names follow the same name rules as types.

### When to use a function
Use a function when a formula is long, is used in more than one place, or has a name that people in the field already recognise, such as a body mass index or a tax rate.

Do not wrap a short comparison in a function. `(EndDate >= StartDate)` is easier to read than `(OnOrAfter(EndDate, StartDate))`, so the plain comparison is the better choice unless the function is used in many places.

A calculation that belongs to one type can already be shared by extending that type, and a rule on a single value can be given a name by extending a simple type. See Extending types. Functions are for the formulas that are left: those used on different elements in different types.

### Not decided yet
The system is built with C# and .NET. Functions provided by the system, and functions written in C#, have not been decided. They will be added when real structures need them.
