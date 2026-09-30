# Brainstorm

Brainstorm is a system that allows users to quickly create lists of items in markdown-like text and then transform them into useful, actionable data. The text can be defined using a brainstorm structure which is also markdown-like and defines the types that can be represented in a brainstorm document. A branstorm structure document is a brainstorm document that defines a structure.
The system provides text to data and data to text where the text and data are associated with the same brainstorm structure. 

## Brainstorm Structure syntax
All declarations are made starting with a token defining what is being declared. A complex item with child elements has the child elements listed below the description which is a description of the type itself to be used by the system to help users understand what the type is for. Since description is always first, it does not need a token to define it. All elements must be pre-defined types or types that are defined in the same brainstrom document: the key at the moment is to keep it simple since this is not meant to be a turning complete programming language, just a way to easily define DDD models.

### Declaring a type
A type is declared with a line starting with t, followed by a space thn the name of the type, a pipe | and the alias. After the declaration the next line contains a description of the type.After the description, the elements of the type are listed with one per line. Each element consists of the alias or name, followed by a space, the name of the element, an optional caption and and optional description. These elements are used for quickly generating UI representations of items and for describing items in brainstorm document hints. The use of squere brackets indicates that the part of the element definition is optional. E.G.
```
t name|alias
    this is a description of the type for use by the system to display help and tips. 
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
By convention, built in simple types have a single character alias, built in complex types have a two character alias staring with t. E.G tt for task and user type aliases have a two or more character alias starting with u. E.G. t sale|us
More formally, we have:
```
t TypeName|alias
    [description]
    s[+][!|!!][|rule]* Name[|caption][|description]
    [alias|typeName[+][!|!!][|rule]* elementName[|caption][|description]]*
    [alias|typeName=(formula)[|rule]* elementName[|caption][|description]]*
```
You will notice that for most of the elements there are two MyThing properties: this is because each element is created with a name and an alias. Aliases allow you to create shortcut names to make producing a brainstorm structure document eaier.

### Long descriptions
A type description is normally a single line. To write a longer description, start it with a line containing only `"""` and end it with another line containing only `"""`. Everything between the two lines is the description and can use full markdown, such as bold text, lists and paragraphs. The indentation of the opening `"""` is removed from every line of the description, so the text is read as if it started at the left margin. For example:
```
t LifeCheck|ulc
    """
    A regular check of my **health, weight and wealth**.

    Use this when you want to:
    * track a few numbers each month
    * compare them against earlier months
    """
    s! Name|Name|What this check is called
    n Weight|Weight (kg)|My weight on the day
```

### Templated and read-only elements
An element is marked as templated by putting `!` straight after its alias or type name, e.g. `s! Name`. Templated elements are the plan. They are copied whenever the item is copied, either on a scheduled date or when a program is published. All other elements are the record, and they start blank in every copy.

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
```
i!!|0..100 Score|Score|A test score between 0 and 100
```
An element can have several rules, and all of them apply. For example, a user name of 3 to 20 lower case letters:
```
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
```
s|/^(home|work|mobile)$/ Label|Label|Where this number reaches me
```
Most people can not read regular expressions, so when a value does not match, the flag shows the description of the element or of its type, never the pattern itself. Write the description as an example of a good value, such as "An email address, such as name@example.com".

Patterns are for experts. To let everyone else use one without reading it, give it a name by extending a simple type. See Extending types.

**Note on regular expressions:** the programming language for the system has not been chosen yet, and regular expressions differ slightly between languages such as Rust and .NET. Until that is decided, patterns should only use features that work the same in both:
* character classes such as `[a-z]`, `\d`, `\w` and `\s`
* the quantifiers `*`, `+`, `?` and `{2,5}`
* the anchors `^` and `$`
* groups `( )` and alternation `|`

Avoid lookarounds, such as `(?=...)`, and backreferences, such as `\1`. .NET supports them, but the standard Rust regex library does not.

#### Conditions
A condition compares an element with other elements on the same item. It is written as a formula between brackets, and the value should make the condition true. For example, a project that should not end before it starts:
```
dd                     StartDate|Starts|When the project starts
dd|(EndDate >= StartDate) EndDate|Ends|When the project ends, on or after the day it starts
```
A condition can use everything a calculated value can (see Calculated values), plus:
* the comparisons `=`, `!=`, `<`, `<=`, `>` and `>=`
* `and` and `or`, to combine comparisons

If an element used by a condition is blank, the condition is not checked. The blank element is flagged on its own if it is expected.

### Calculated values
A calculated value is worked out from other elements on the same item. It is declared by putting `=` straight after the alias, followed by a formula between brackets. The brackets mean the formula can contain spaces. For example:
```
c RiskType
    Financial
    Reputational
    HealthAndSafety
    Legal

t ProjectRisk|upr
    The possibility that an action, event or decision leads to a bad outcome, and what it would cost.
    s+!                           Name|Name|A short name for the risk
    i|0..100                      Likelihood|Likelihood (%)|How likely the risk is to happen, from 0 to 100%
    n|0..                         Impact|Impact|What it would cost if it happened
    RiskType                      RiskType|Risk Type|The kind of harm the risk would cause, which shapes what its impact means
    n=(Likelihood / 100 * Impact) RiskScore|Risk Score|The cost of the risk once its likelihood is taken into account
```
Watch the units in a formula. Likelihood is a percentage from 0 to 100, so it is divided by 100. Without that, a 50% chance of a 10,000 loss would give a risk score of 500,000 instead of 5,000.

A formula can use:
* numbers
* the names of other elements on the same item, including other calculated values
* `+`, `-`, `*` and `/`
* brackets, to control the order things are worked out in

The rules for calculated values are:
* A calculated value is always read-only. Users never write it in a document. If a document gives it a value, the value is ignored and flagged.
* The result has the type of the alias. `n=` gives a decimal, and `i=` rounds to the nearest whole number.
* If a value the formula uses is blank or can not be understood, or the formula divides by zero, the result is blank. It is not flagged, because the value it depends on is flagged already.
* A calculated value is never copied. It is worked out again in each copy, so `!`, `!!` and `+` are not used with it.
* Rules can follow the formula. For example, `n=(Likelihood / 100 * Impact)|..10000` flags any risk score over 10,000.
* A formula that depends on itself, directly or through other calculated values, is reported as a problem in the structure document.
* Calculated values are left out when a brainstorm is written out as a brainstorm document. A brainstorm document is a working document that only holds what people type. Reports can include calculated values.

Calculations over lists, such as totals, are not supported yet. They may be added later using something like C# LINQ expressions.

## Built-in types
Built in types are the framework for creaing and defining the structures you want to work with. They can not be modified, but they can be extended (see below). These types provide data essential for some of the built-in services in the system such as user logins, contact details and task management. These types often need no modification to use and often are not to be used in Brainstorm documents. For example, the user that is set for a tasks owner or creator will be the user that imports the document.

### Choices
A choice is a list of text values that are tied to a named choice. For example, we could have a choice called Months that allow a user to select from a list of dates. Choices are defined as follows:
```
c choiceName
    [choiceValue]*
```
Because many scenarios have a large number of choice-style values, choices do not support aliases and must be explicitly named. 

To define the Months choice for example, we have the following:
```
c Months
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
```
t MyType|umt
    An example type that offers a chice of Months
    Months TheMonth|The Month|The month selected for fun
```

### Built-in simple types
The following types are built in and the names are reserved:
* t Integer|i
    A whole number
* t DateTime|d
    A date and time
* t Time|dt
    A time only date object
* t Date|dd
    A date only date object
* t String|s
    Text value
* t Decimal|n
    A decmal number with a whole and fractional component expressed in decimal places
* t Boolean|b
    A true or false value: items of this type are either true or false. Missing values are assumed to be false

### Built-in complex types
* t List<T>|l:x
    A generic list of objects where x is the name or alias of the objects that the list holds
* t TaskItem|tt
    A task object that we can use in kanban boards
**Definition**
```
t TaskItem|tt
    A TaskItem is anything that needs to have action taken to implement some desired outcome. This can be used in a todo list, a list of things that must be done to get controls in place for a risk, a projects kanban board, etc.    
    s!!+ Name|Task Name|A short name to identify the task without a full description
    s!+ Description|Description|A full description of what the task entails, what needs to be done and any other information pertinent to the execution and completion of the task
    d CreatedDate|Created Date|The date and time the task was created
    TaskStatus Status|Status|The status of the task
    i Priority|Priority|When a task appears in a list priority can be used to sort them.  
    d DueDate|Due Date|The date and time the task should be done by
    tu CreatedBy|Created By|The user that created the task
    d CompletedOn|Completed On|The date and time the task was done
    l:tt SubTasks|Sub-tasks|The task has been broken down into smaller tasks and can be considered complete when the sub tasks are all done.
```
* t ScheduledTask|tst
    A task that repeats on its own schedule. It is always a root item and is never placed inside another item.
**Definition**
```
x ScheduledTask:TaskItem|tst
    A task that is created again on a schedule, such as a weekly visit or a monthly bill payment.
    ts Schedule|Repeats|When and how often the task is created again
```
* t Contact|tc
    A contact with contact details such as a phone number, email address, etc. 
    **Definition**
```
t contact|tc
    An contact is a party that has contact details
    s Name|Name|The first name of the contact 
    s Surname|The surname of the contact
    s CompanyName|The name of the company if the contact is a contact person in a company
```
* t ContactDetail|tcd
    A single detaial about a contact, for example, the phone number or email address.
    ContactDetailType Type|Contact Detail Type|One of the predefined contact types in the ContactDetailType choice.
    s Name|Label|An optional note such as "home" or "after hours"
    s Value|Value|The actual contact detail value
    public bool IsPrimary|Is Primary|The one to use first when there is more than one of this type.
* t User|tu
    Another user of the system which also has contact details.
    **Definition**
```
 t User|tu
    A user of the system
    s Name|Login|The user login name (usually an email address)
    cu Contact|Contact|The person or party that the user represents    
    dt CreatedOn|Created On|The date the user record was created
    b IsActive|Is Active|True if the user is actively using the system.
```
* t Goal|tg
* t Risk|tr
* t Control|trc
* t Assessment|ta
* t Schedule|ts
    A schedule says when a new copy of an item should be created. A root item with a Schedule element is repeated on that schedule. See Repeating items.
    s Name|Name|A short name for the schedule, e.g. "Monthly check-in"
    Frequency Repeats|Repeats|How often a new copy is created
    i Every|Every|Repeat every N periods: 1 is every month, 2 is every other month. Missing means 1.
    dd StartDate|Starts|The date of the first copy. It also sets the day, e.g. starting on the 5th means monthly on the 5th.
    dd EndDate|Ends|Optional. No copies are created after this date.
    dt DueTime|Due At|Optional time of day the copy is due
    dd NextDate|Next Date|Set by the system: the date the next copy will be created

### Built-in choices
ContactDetailType - used in contact detail to select a type. 
**definition**
```
c ContactDetailType
    Email
    PhoneNumber
    WorkPhone
    MobilePhone
    SocialMediaUrl
    Address
    Other
```
Frequency is the choice for setting up schedules
```
c Frequency
    Once
    Daily
    Weekly
    Monthly
    Quarterly
    Yearly
```
TaskStatus is the set of statuses that a task can be in: used for the kanban board and tracking projects
```
c TaskStatus
    ToDo
    InProgress
    InReview
    Blocked
    OnHold
    Done
    Dropped
```

### Extending types
It can often be the case that you want some new peice of data on an existing type. For example, in the Task you may want to add a new element to allow you to delegate that task to a contact. In this case it is better, in fact essential, to extend the task rather than copy and paste it's definition, so that the system can still recognise the item as a task. Here is how you would implement such a change:
```
x DelegatedTask:TaskITem|udt
    A task that can be delegated to a contact: useful for team management and sharing tasks.
    tc DelegatedTo|Delegate|The person that is responsible for the task completion.
```
The declaration of this is similar to the declaration of a normal type, but starts with x and must have a colon between the name of the new type and the type that it extends. The description of this new type will be used instead of the description of the type that it extends. All other elements can be appended in the same manner as they are declared in a normal complex type. Any duplicates are simply ignored.

Simple types can be extended too. This is how a rule is given a name, so that people can use it without having to read or write it. The rules are written after the alias, each starting with a pipe. For example, an email address is text that matches a pattern:
```
x EmailAddress:s|em|/^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$/
    An email address, such as name@example.com
```
Anyone can now use `em` like any other type, without knowing the pattern:
```
em+ Email|Email Address|Where to send reminders
```
An extended simple type only adds rules and a description. It has no elements. If an element that uses it adds rules of its own, both sets of rules apply.

### Name rules
For a name to be valid, these simple rules must be followed:
* Names must start with a letter or underscore (_).
* Names can contain alpha-numeric characters and the underscore (_).

## Brainstorm documents
A brainstorm document is a text document, similar to markdown, that allows us to quickly list items in meaningful ways, without having to do data capture on some form or spreadsheet. This saves us from having to navigate between controls on pages, click save a hundred times and wait for excel to think when we want to add an item.
Some use cases are quickly creating action plans, identifying risks and controls, setting goals and planning how to achieve them. For example given a type goal defined as follows:
```
t Goal|ug
    A goal is an aim, purpose or desired result that you work hard to achieve
    s Name|Goal|A short name to remind me of the goal
    s Description|Description|A full description of what I am trying to achieve
    d TargetDate|Target Date|The date by which I will achieve this goal
    s Reason|Reason|It's good to understand why I am chasing a goal and whether it is worth the effort. This is a reminder.
    l:tt PlanTasks|Plan|A list of the tasks that I need to do in order to achieve this goal.
```
I can now create a goal with a list of tasks easily as follows:
```
Goal: Book a holiday
    PlanTasks:
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
* A complex element, such as a Schedule, can use shorthand after its colon. Its other elements are indented one level further below it.
* Values on the line follow the order the elements are declared in the type, starting at Name. The type description belongs to the type, not to each item, so it never takes a position.
* Only simple elements take a position: text, numbers, dates, times, true/false values and choices. Lists and complex elements are skipped when counting positions and are always written below the line.
* Leave a position empty to skip it. For example, `a||c` skips the second value.
* Any element can be written on its own indented line below, as the element name, a colon, then the value.
* Values on the shorthand line can not contain a pipe. Write such a value on its own line below instead.

Because an extended type adds its elements after those of the type it extends, shorthand written for the original type still works for the extended one.

Using the Goal type above, the values on the line are Name, Description, TargetDate and Reason, in that order. Here the Reason is written below the line instead:
```
Goal: Book a holiday|Sun, sea and no laptop|2027-03-01
    Reason: I haven't had a proper break in two years
    PlanTasks:
    * Find a sunny destination wich fits into my budget
    * find the best flights
    * book a hotel
```

### When problems are reported
Import never fails. A value the system can not understand, such as `Weight: about 80`, an expected value that is missing, or a value that breaks a rule, is kept as written and flagged so it can be fixed later in the document or in structured mode. While writing, flags are quiet hints. They are pointed out again when an item is marked as done, but nothing is blocked.

Publishing and scheduling are stricter, because other people and future copies depend on the plan being complete. A program can not be published while its plan has problems, and a schedule does not create a copy while the original's plan has problems. The author is told what needs fixing.

The plan is everything that gets copied: templated elements, lists, and the templated elements of each item in those lists. Record elements, such as Weight in a LifeCheck, are not checked, because they start blank in every copy anyway.

# The Brainstorm application
The application supports editing brainstorm structure documents and uses these documents to generate a UI based on the defined types in the brainstorm structure. A user has a number of brainstorms available which can be viewed as a document/text, or as structured objects (in structured mode) where the UI is based on the structure document that the brainstorm document uses. The structure document is not used to generate code, but instead to create an in-memory object model that is tolerant of user errors. 
When a user is viewing their brainstorm in structured mode the layout is driven by the model defined in the brainstorm structure document.
Out of the box, all strucuters inherit tasks, goals, risks, controls, contacts, contact details, schedule, assesment and assessed items which are pre-defined. Goals, risks, controls and assessments are included in the structure document when a new brainstorm structure document is created, allowing users to remove these items. Tasks, contacts, contact details and users are not deletable. This is because tasks are the most meaningful thing in a brainstorm and are a first class structure. Tasks must be supported by a kanbas board in the UI.
The system allows users to keep a catalog of Brainstorm structure documents (on disk or in a database) and a catalog of brainstorms. Some brainstorms, like assessment-style brainstorms must support repeating so that the assessed items can be measured and compared against historic values. For example, give an assessment style structure as follows:
```
t LifeCheck|ulc
    A regular check of my health, weight and wealth
    s+! Name|Name|What this check is called
    n! TargetWeight|Target Weight (kg)|The weight I am aiming for
    ts Schedule|Schedule|How often I do this check
    n+ Weight|Weight (kg)|My weight on the day
    i RestingHeartRate|Resting Heart Rate|Beats per minute first thing in the morning
    n NetWorth|Net Worth|Assets minus debts
    l:tt+ Steps|Steps|The things I do to complete the check
```

### Repeating items
Structures are a way of knowing what tasks need to be done and why. Repeating an item is how the system creates the next round of tasks.

A schedule only counts on a root item, which is an item that is not inside another item. To repeat a single task, use a ScheduledTask. A schedule on a nested item is ignored.

The item you write is the original. The system creates a copy of it on each scheduled date, and the same rules are used when a program is published:
* Only templated elements, marked with `!` or `!!`, are copied. All other elements start blank, so they can be filled in on the day and compared with earlier copies.
* Elements marked with `!!` are locked in the copy: simple values can not be changed, lists can not have items added or removed, and complex elements can not be deleted.
* Lists are always copied, whether or not they are marked. Each item in a list is created using the `!` and `!!` marks of its own type. A list of simple values, such as `l:s`, is copied as it is.
* A templated complex element is copied the same way: its own elements follow the `!` and `!!` marks of its type.
* If Name is templated, a scheduled copy gets the date added to it, e.g. "Life check 2026-10-01".
* Every task in the copy, at any depth, has the Status ToDo and a DueDate of the scheduled date, plus the DueTime if one is set.
* The Schedule is not templated, so the copy has no schedule and only the original repeats. A copy of a ScheduledTask is a plain TaskItem.
* No copy is created while the original's plan has problems. See When problems are reported. A scheduled date that passes while the plan has problems is skipped. It is not caught up automatically once the plan is fixed.

A user can also create a copy by hand, for example to make up for a skipped date. The user picks the date of the copy, which defaults to today, and the copy is created using the same rules as a scheduled copy. The plan must have no problems, just as for a scheduled copy. Creating a copy by hand does not change the schedule.

For example, using the LifeCheck type above, this is the original a user writes:
```
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
```
LifeCheck: Life check 2026-10-01
    TargetWeight: 78
    Steps:
    * Weigh myself before breakfast
    * Take my resting heart rate
    * Update the net worth spreadsheet
```
A single task that repeats is written as a ScheduledTask. This one is created every other Wednesday:
```
ScheduledTask: Visit mom|Pop round for tea and help with the garden
    Schedule: Every other Wednesday|Weekly|2|2026-09-30
        DueTime: 9:00 am
```