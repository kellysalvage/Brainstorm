# Brainstorm

Brainstorm is a system that allows users to quickly create lists of items in markdown-like text and then transform them into useful, actionable data. The text can be defined using a brainstorm structure which is also markdown-like and defines the types that can be represented in a brainstorm document. A branstorm structure document is a brainstorm document that defines a structure.
The system provides text to data and data to text where the text and data are associated with the same brainstorm structure. 

## Brainstorm Structure syntax
All declarations are made starting with a token defining what is being declared. A complex item with child elements has the child elements listed below the description which is the default first item. Since description is always first, it does not need a token to define it. All elements must be pre-defined types or types that are defined in the same brainstrom document: the key at the moment is to keep it simple since this is not meant to be a turning complete programming language, just a way to define simple DDD models with no behavior and validation.

### Declaring a type
A type is declared with a line starting with t, followed by a space thn the name of the type, a pipe | and the alias. After the declaration, all elements are indented, starting with the optional description. Each element consists of the alias or name, followed by a space, the name of the element, an optional caption and and optional description. These elements are used for quickly generating UI representations of items and for describing items in brainstorm document hints. The use of squere brackets indicates that the part of the element definition is optional. E.G.
```
t name|alias
    description: this is the default first element
    s Name|Name|Name must always be after the description.
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
```
By convention, built in simple types have a single character alias, built in complex types have a two character alias staring with t. E.G tt for task and user type aliases have a two or more character alias starting with u. E.G. t sale|us
More formally, we have:
```
t TypeName|alias
    [description]
    s Name[|caption][|description]
    [alias|typeName elementName[|caption][|description]]*
```
You will notice that for most of the elements there are two MyThing properties: this is because each element is created with a name and an alias. Aliases allow you to create shortcut names to make producing a brainstorm structure document eaier.

## Built-in types
Built in types are the framework for creaing and defining the structures you want to work with. They can not be modified, but complex types can be extended (see below). These types provide data essential for some of the built-in services in the system such as user logins, contact details and task management. These types often need no modification to use and often are not to be used in Brainstorm documents. For example, the user that is set for a tasks owner or creator will be the user that imports the document.

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
* t TaskItem|t
    A task object that we can use in kanban boards
**Definition**
```
t TaskItem|tt
    A TaskItem is anything that needs to have action taken to implement some desired outcome. This can be used in a todo list, a list of things that must be done to get controls in place for a risk, a projects kanban board, etc.
    s Name|Task Name|A short name to identify the task without a full description
    s Description|Description|A full description of what the task entails, what needs to be done and any other information pertinent to the execution and completion of the task
    d CreatedDate|Created Date|The date and time the task was created
    s Status { get; set; } = string.Empty;
    i Priority|Priority|When a task appears in a list priority can be used to sort them.  
    dt DueDate { get; set; }
    dt CreatedOn { get; set; }
    tu CreatedBy|Created By|The user that created the task
    dt CompletedOn { get; set; }
    l:tt SubTasks|Sub-tasks|The task has been broken down into smaller tasks and can be considered complete when the sub tasks are all done.
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

### Extending types
It can often be the case that you want some new peice of data on an existing type. For example, in the Task you may want to add a new element to allow you to delegate that task to a contact. In this case it is better, in fact essential, to extend the task rather than copy and paste it's definition, so that the system can still recognise the item as a task. Here is how you would implement such a change:
```
x DelegatedTask:TaskITem|udt
    A task that can be delegated to a contact: useful for team management and sharing tasks.
    tc DelegatedTo|Delegate|The person that is responsible for the task completion.
```
The declaration of this is similar to the declaration of a normal type, but starts with x and must have a colon between the name of the new type and the type that it extends. The description of this new type will be used instead of the description of the type that it extends. All other elements can be appended in the same manner as they are declared in a normal complex type. Any duplicates are simply ignored.

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
Goal:Book a holiday
    PlanTasks
    * Find a sunny destination wich fits into my budget
    * find the best flights
    * book a hotel
    * get travel insurance
    * hire a car or book transfers to the destination from the airport
    * look into local acivities and excursion
```