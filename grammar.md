# Brainstorm grammar

This is the formal grammar of the Brainstorm language. README.md explains what each part means; this document says exactly how each part is written. It is for people building tools that read Brainstorm documents, including the Brainstorm parser itself.

It covers structure documents (`.bss`), the formulas inside them, and brainstorm documents (`.bsd`).

## Notation
The grammar uses the notation of the XML specification:

| Notation | Meaning |
|---|---|
| `A ::= B` | A is written as B |
| `"t"` | exactly these characters |
| `A B` | A followed by B |
| `A \| B` | A or B |
| `A?` | A is optional |
| `A*` | zero or more of A |
| `A+` | one or more of A |
| `( )` | a group |
| `[a-z]` | one character in the range |
| `[^\|]` | any one character except those listed |

## How a structure document is read
A structure document is read one line at a time, in two steps:
1. **Finding the definitions.** The front matter is read as YAML. Only the lines inside `bss` fences are read with the grammar below. Every other line is documentation.
2. **Reading the definitions.** Each declaration starts on a line that is not indented. Its body is the indented lines below it, up to the next line that is not indented.

A line that does not match the grammar is flagged and skipped, and reading carries on with the next line. One mistake never hides the rest of the document.

A line is indented when it starts with one or more spaces or tabs. The lines of a body should line up with each other. A line indented differently from the line above it is still read, but the editor points it out, because it may have been meant to belong to the line above. A line is only refused when its indent makes its meaning ambiguous.

Names are not case sensitive. `Goal`, `goal` and `GOAL` are the same name, wherever a name is used.

## The document
```ebnf
StructureDocument ::= FrontMatter? ( Fence | DocumentationLine )*

FrontMatter       ::= "---" EOL YamlLine* "---" EOL
YamlLine          ::= TextLine
Fence             ::= FenceOpen FenceLine* FenceClose
FenceOpen         ::= Backticks "bss" Space* EOL
FenceClose        ::= Backticks Space* EOL
Backticks         ::= "```" "`"*
```
* The front matter must start on the first line. It ends at the first line that is only `---`, and the lines between are read by a YAML reader rather than by this grammar.
* A fence is closed by a line with at least as many backticks as opened it.
* Lines end with a line feed, or a carriage return and a line feed.

### Versions
The front matter is YAML. Its `versions` key holds a map from version keys to change notes:
```ebnf
VersionKey    ::= VersionNumber " " Date
VersionNumber ::= Digit+
```
The change notes are text, one note per line. A note that starts with `nb:` tells users something they need to change in their own documents.

## Declarations
```ebnf
FenceLine   ::= Declaration | BlankLine

Declaration ::= TypeDeclaration
              | ExtensionDeclaration
              | ChoiceDeclaration
              | FunctionDeclaration
```
Blank lines can appear anywhere between declarations and between the lines of a body.

### Types
```ebnf
TypeDeclaration      ::= "t" Space+ Name "|" Name EOL
                         Description?
                         Element*

ExtensionDeclaration ::= "x" Space+ Name ":" TypeRef "|" Name Rule* EOL
                         Description?
                         Element*
```
The first name is the type's name, and the name after the pipe is its alias. The alias is required, and can be the same as the name. In an extension, the type after the colon is the type being extended. Only an extension of a simple type, such as `x EmailAddress:s|em|/.../`, can have rules on its first line.

### Descriptions
```ebnf
Description     ::= DescriptionLine+
DescriptionLine ::= Indent ">" ( " " Text )? EOL
```
* A description comes straight after the declaration line. Each of its lines starts with `>`, so it can never be mistaken for an element or a choice value, because names never start with `>`.
* The `>` and one space after it are removed, and the lines are joined into one Markdown text. A line with only `>` separates two paragraphs.
* A type, extension or function without a description is flagged. A choice's description is optional.

### Elements
```ebnf
Element     ::= Indent ElementHead Space+ Name ( "|" Caption )? ( "|" ElementDescription )? EOL

ElementHead ::= TypeRef Marks? DefaultValue? Rule*
              | TypeRef Calculation Rule*

TypeRef     ::= ( "l:" | "r:" )* Name
Marks       ::= "+" | Lock | "+" Lock | Lock "+"
Lock        ::= "!" | "!!"

DefaultValue ::= "=" ( QuotedText | Word )
Calculation  ::= "=" "(" Formula ")"

Caption            ::= [^|]+
ElementDescription ::= Text
```
* The head is everything before the first space that is not inside brackets, quotes or a pattern. That is why a formula, a quoted default or a pattern can contain spaces.
* A calculated value has no marks and no default.
* With one part after the name, it is the caption. With two, they are the caption and the description.
* The description is the rest of the line, so it can contain pipes.

### Rules
```ebnf
Rule      ::= "|" ( Range | Pattern | Condition )

Range     ::= Bound? ".." Bound?
            | Bound
Bound     ::= Number | Date | Time | DateTime

Pattern   ::= "/" ( "\" Char | [^/\] )* "/"
Condition ::= "(" Formula ")"
```
* A pattern ends at the first `/` that is not written as `\/`.
* Inside a pattern, only the regular expression features listed in README.md are allowed.

### Choices
```ebnf
ChoiceDeclaration ::= "c" Space+ Name EOL
                      Description?
                      ChoiceValue*

ChoiceValue       ::= Indent Name ( "|" Caption )? ( "|" ValueDescription )? EOL
ValueDescription  ::= Text
```
* A value's caption and description work like an element's: with one part after the name it is the caption, and the description is the rest of the line.
* In a brainstorm document, a value can be written as its name or its caption.

### Functions
```ebnf
FunctionDeclaration ::= "f" Space+ TypeRef Space+ Name "(" Parameters? ")" EOL
                        Description?
                        Indent "=" Space* Formula EOL

Parameters          ::= Parameter ( Space* "," Space* Parameter )*
Parameter           ::= TypeRef Space+ Name
```

## Formulas
Formulas are used in calculated values, conditions and functions:
```bss
n=(Likelihood / 100 * Impact)                     RiskScore|...
dd|(EndDate >= StartDate)                         EndDate|...
n=(sum(Clients.Deals.Value, Stage = Won))         Won|...
    = Percent / 100 * Amount
```
A formula is read as a list of tokens rather than as lines, so it is described in two parts: the tokens, and how they are put together.

### Tokens
```ebnf
Token      ::= Number | Name | Operator | "(" | ")" | "," | "."
Operator   ::= "+" | "-" | "*" | "/" | "=" | "!=" | "<" | "<=" | ">" | ">="
```
* Spaces can go between tokens, and are ignored. They can not go inside a number, a name or a path, so `Clients.Deals.Value` has no spaces.
* `and`, `or` and `not` are words of the formula language, not names, in any case. An element called `And`, `Or` or `Not` could not be used in a formula, so the name is flagged where it is declared.
* A formula has no text and no dates of its own. It works with numbers, the values of elements, and choice values.

### Expressions
Each rule is one level of precedence, from the loosest to the tightest. So `a or b and c` means `a or (b and c)`, `not a and b` means `(not a) and b`, and `1 + 2 * 3` means `1 + (2 * 3)`.
```ebnf
Formula    ::= Or
Or         ::= And ( "or" And )*
And        ::= Not ( "and" Not )*
Not        ::= "not"? Comparison
Comparison ::= Sum ( CompareOp Sum )?
CompareOp  ::= "=" | "!=" | "<" | "<=" | ">" | ">="
Sum        ::= Product ( ( "+" | "-" ) Product )*
Product    ::= Unary ( ( "*" | "/" ) Unary )*
Unary      ::= "-"? Primary
Primary    ::= Number
             | Call
             | Path
             | "(" Formula ")"

Call       ::= Name "(" ( Formula ( "," Formula )* )? ")"
Path       ::= Name ( "." Name )*
```
* Operators of the same level are worked out from left to right, so `10 - 2 - 3` is 5.
* `not` applies to the comparison straight after it, so `not Stage = Won` means `not (Stage = Won)`. It can not be repeated, because `not not` is never needed.
* A comparison compares two values only. `1 < x < 5` is not allowed; write `1 < x and x < 5`.
* A name followed by `(` is a call. Otherwise it is a path, and a path of one name is just a name.

### What the grammar does not decide
The grammar only says how a formula is written. These rules are checked once the formula has been read, against the structure:
* **Calls.** A call is one of the totals (`sum`, `count`, `average`, `min`, `max`, `stddeviation`) or a function declared in the structure.
* **Paths.** A path of more than one name can only be the first value given to a total, and it can only go down through the item's own lists.
* **Names.** A name is looked up on the item the formula belongs to. In a total's condition, it is first looked up on each item at the end of the path. In a function, only the function's parameters can be used.
* **Choice values.** When a choice is compared with a name, as in `Stage = Won`, the name is read as one of the choice's values.
* **Types.** `+`, `-`, `*` and `/` work on numbers. A comparison gives true or false, and `and`, `or` and `not` work on true or false. A true or false element, or a function that gives true or false, can be used on its own as a condition, so formulas never need the values true and false.

## Brainstorm documents
A brainstorm document is read in three steps:
1. **The front matter** is read as YAML, to find the structure.
2. **The shape.** Each line is read as one of the kinds of line below, and the lines are put into a tree by their indents.
3. **The meaning.** The tree is read again with the structure, which says which names are types and elements, and how each value is read.

The grammar covers the first two steps. The third depends on the structure, so it is described in words at the end of this part.

### The document
```ebnf
BrainstormDocument ::= FrontMatter? ( Heading | Item | Note | NoteFence | BlankLine )*

Heading            ::= "#" "#"? "#"? "#"? "#"? "#"? Space+ Text EOL
NoteFence          ::= Backticks Text? EOL TextLine* Backticks Space* EOL
```
* The front matter's `structure` key holds a file path, a web address with an optional `@` and version number, or `built-in`.
* A heading starts a section. Sections hold the items and notes below them, up to the next heading of the same or a higher level.
* A note is any other text. A fenced block is always part of a note, so nothing inside it is read as an item.

### Lines
Every line is one of four kinds:
```ebnf
LabelledLine ::= Indent? Name Space* ":" ( Space* Value )? EOL
ListItemLine ::= Indent? "*" Space+ ( Name Space* ":" Space* )? Value EOL
BlankLine    ::= Space* EOL
TextLine     ::= [^\n]* EOL
```
* A labelled line is a name, a colon and an optional value. Only the first colon counts, so a value can contain colons, as in `DueTime: 9:00 am`.
* A list item line starts with `*` and a space. `**bold**` has no space after the first `*`, so it is text.
* Spaces around the colon and at the ends of a value are ignored.

### Items and the tree
```ebnf
Item ::= LabelledLine Body?
Body ::= ( LabelledLine Body? | ListItemLine Body? | BlankLine )+
```
The grammar can not show indents, so these are the rules for putting lines into a tree:
* **Root items.** A labelled line that is not indented starts an item, when its name is a type or an alias. When it is not, the line is flagged as a possibly mistyped type, as in `Gaol: Book a holiday`, and kept as a note.
* **Bodies.** The lines below a line that are indented further belong to it, up to the next line that is indented the same or less.
* **Lists.** A list element is a labelled line with no value. Its items are the list item lines below it, at the same indent or further. The samples write them at the same indent, as in Markdown.
* **Lining up.** Lines in the same body should line up. A line that does not is still read, and the editor points it out. A line is only refused when its indent makes its parent ambiguous.
* **Stray lines.** In a body, a line that is neither a labelled line nor a list item is kept as a note and flagged.

### Values
```ebnf
Value ::= Field ( "|" Field )*
Field ::= [^|\n]*
```
Whether a value is split at its pipes depends on what it sets:
* **An item or a complex element**, such as `Goal: Book a holiday|Sun and sea|2027-03-01` or `Schedule: Gym|Weekly||2026-10-05`, is shorthand. The fields fill its simple elements in the order they are declared. An empty field skips a position.
* **A simple element** takes the whole value, pipes included. This is how to write a value that contains a pipe.
* **A list item** of a list of simple values, such as `l:s`, takes the whole value too.

A list item can start with a type or alias and a colon, as in `* udt: Book the venue`. When the name is not a type that the list can hold, the whole line is the item's value, and it is not flagged. So `* Achieved: I reached 3rd dan` is an item whose text starts with "Achieved".

### Simple values
How a value is read depends on the type of its element. These are the forms each type accepts:
```ebnf
Boolean      ::= "true" | "false"
DocumentTime ::= Time ( Space* ( "am" | "pm" ) )?
DocumentDate ::= Date | RelativeDate
DocumentDateTime ::= Date ( "T" | " " ) DocumentTime | RelativeDate
RelativeDate ::= "+" Digit+ Space* Unit
Unit         ::= "d" | "w" | "m" | "y"
               | "day" | "days" | "week" | "weeks" | "month" | "months" | "year" | "years"

Link         ::= "[[" LinkName ( "/" LinkName )* "]]"
LinkName     ::= [^/\]\n]+

Media        ::= "!"? "[" MediaCaption "]" "(" Target ")"
               | Target
MediaCaption ::= [^\]\n]*
Target       ::= [^ )\n]+
```
* Text can use inline Markdown and tags. Numbers use `Number`.
* Dates follow ISO 8601, year first.
* A choice value is one of the choice's names or captions.
* Words in values, such as `true`, `pm`, `weeks` and `Mon`, are not case sensitive.
* A link leaves out the tags in the name it links to. An item name containing `/`, `[` or `]` is pointed out where it is written, because no link can reach it.
* A link to a user can name them, as in `[[Sam Jones]]`, or give their email address, as in `[[sam@example.com]]`.
* A value that does not match its type is kept as written and flagged.

### Tags
```ebnf
Tag ::= "#" Name
```
* A tag can be in any text: names, text values, headings and notes.
* It starts at the beginning of the text or after a space, so `C#` is not a tag. A heading's `#` is followed by a space, so it is not a tag either.
* A tag belongs to the item whose text it is in, and not to the items inside it.

### Schedule days
The `On` element of a Schedule is text, read by the scheduler in this form:
```ebnf
Days     ::= DaySpec ( Space* "," Space* DaySpec )*
DaySpec  ::= ( Position Space+ )? DayName | "weekdays" | "weekends"
Position ::= "1st" | "2nd" | "3rd" | "4th" | "last"
DayName  ::= "Mon" | "Tue" | "Wed" | "Thu" | "Fri" | "Sat" | "Sun"
           | "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" | "Saturday" | "Sunday"
```

### What the grammar does not decide
These are decided with the structure, once the tree has been read:
* **Types and elements.** Whether a name is a type, an alias or an element of the item's type. An element that only an extension has turns a list item into that extension.
* **Unknown elements.** A labelled line whose name is not an element of the item's type is kept as written, with its body, and flagged.
* **Shorthand.** Which elements the fields fill. Lists, complex elements and calculated values take no position.
* **Values.** Whether each value suits its element's type, rules and conditions.
* **Links.** Which item, user or contact each link names.

## Words and values
```ebnf
Name       ::= [A-Za-z_] [A-Za-z0-9_]*
Word       ::= [^ \t|"(] [^ \t|]*
QuotedText ::= '"' [^"]* '"'

Number     ::= "-"? Digit+ ( "." Digit+ )?
Date       ::= Digit Digit Digit Digit "-" Digit Digit "-" Digit Digit
Time       ::= Digit Digit? ":" Digit Digit ( ":" Digit Digit )?
DateTime   ::= Date "T" Time
Digit      ::= [0-9]
Char       ::= [^\n]

Text       ::= [^\n]+
TextLine   ::= [^\n]* EOL
Indent     ::= [ \t]+
Space      ::= " " | "\t"
BlankLine  ::= Space* EOL
EOL        ::= "\n" | "\r\n"
```
* A default value with spaces or a pipe is written in double quotes.
* Values such as `today`, `now`, `me` and `+1w` are words. What they mean is decided by the type of the element, not by the grammar.
