# Embedded Ruby (ERB) Template Syntax

ERB is a templating language based on Ruby. Puppet can evaluate ERB templates with the template and inline_template functions.

This page covers how to write ERB templates. See Templates for information about what templates are and how to evaluate them.

Note: If you’ve used ERB in other projects, it might have had different features enabled. This page only describes how ERB works in Puppet.

###ERB Structure and Syntax
```
<%# Non-printing tag ↓ -%>
<% if @keys_enable -%>
<%# Expression-printing tag ↓ -%>
keys <%= @keys_file %>
<% unless @keys_trusted.empty? -%>
trustedkey <%= @keys_trusted.join(' ') %>
<% end -%>
<% if @keys_requestkey != '' -%>
requestkey <%= @keys_requestkey %>
<% end -%>
<% if @keys_controlkey != '' -%>
controlkey <%= @keys_controlkey %>
<% end -%>

<% end -%>
```

An ERB template looks like a plain-text document interspersed with tags containing Ruby code. When evaluated, this tagged code can modify text in the template.

Puppet passes data to templates via special objects and variables, which you can use in the tagged Ruby code to control the templates’ output.

###Tags
ERB has two tags for Ruby code, a tag for comments, and a way to escape tag delimiters.

* `<%= EXPRESSION %>` — Inserts the value of an expression.
 * With `-%>` — Trims the following line break.
* `<% CODE %>` — Executes code, but does not insert a value.
 * With `<%-` — Trims the preceding indentation.
 * With `-%>` — Trims the following line break.
* `<%# COMMENT %>` — Removed from the final output.
 * With `-%>` — Trims the following line break.
* `<%% or %%>` — A literal `<%` or `%>`, respectively.

Text outside a tag becomes literal text, but it is subject to any tagged Ruby code surrounding it. For example, text surrounded by a tagged if statement only appears in the output if the condition is true.

###Expression-Printing Tags

`<%= @fqdn %>`

An expression-printing tag inserts values into the output. It starts with an opening tag delimiter and equals sign (`<%=`) and ends with a closing tag delimiter (`%>`). It must contain a snippet of Ruby code that resolves to a value; if the value isn’t a string, it will be automatically converted to a string using its to_s method.

For example, to insert the value of the `$fqdn` and `$hostname` facts in an Apache config file, you could do something like:
```
ServerName <%= @fqdn %>
ServerAlias <%= @hostname %>
```

###Space Trimming
You can trim line breaks after expression-printing tags by adding a hyphen to the closing tag delimiter.

`-%>` — If the tag ends a line, trim the following line break.

### Non-Printing Tags

`<% if @broadcastclient == true %> ...text... <% end %>`

A non-printing tag executes the code it contains, but doesn’t insert a value into the output. It starts with an opening tag delimiter (<%) and ends with a closing tag delimiter (%>).

Non-printing tags that contain iterative or conditional expressions can affect the untagged text they surround.

For example, to insert text only if a certain variable was set, you could do something like:

```
<% if @broadcastclient == true -%>
    broadcastclient
    <% end -%>
```

Non-printing code doesn’t have to resolve to a value or be a complete statement, but the tag must close at a place where it would be legal to write another statement. For example, you couldn’t write:
```
<%# Syntax error: %>
<% @servers.each -%>
# some server
<% do |server| %>server <%= server %>
<% end -%>
```

You must keep `do |server|` inside the first tag, because you can’t insert an arbitrary statement between a method call and its required block.

###Space Trimming
You can trim whitespace surrounding a non-printing tag by adding hyphens (-) to the tag delimiters.

* `<%-` — If the tag is indented, trim the indentation.
* `-%>` — If the tag ends a line, trim the following line break.

###Comment Tags

`<%# This is a comment. %>`

A comment tag’s contents do not appear in the template’s output. It starts with an opening tag delimiter and a hash sign (`<%#`) and ends with a closing tag delimiter (`%>`).

###Space Trimming
You can trim line breaks after comment tags by adding a hyphen to the closing tag delimiter.

`-%>` — If the tag ends a line, trim the following line break.

###Literal Tag Delimiters

If you need the template’s final output to contain a literal `<%` or `%>`, you can escape them as `<%%` or `%%>`.

###Accessing Puppet Variables
Templates can access Puppet’s variables. This is the main source of data for templates.

An ERB template has its own local scope, and its parent scope is set to the class or defined type that evaluates the template. This means a template can use short names for variables from that class or type, but it can’t insert new variables into it.

There are two ways to access variables in an ERB template:

* `@variable`

* ```scope['variable'] (and its older equivalent, scope.lookupvar('variable')) ```

###@variable

All variables in the current scope (including global variables) are passed to templates as Ruby instance variables, which begin with “at” signs (`@`). If you can access a variable by its short name in the surrounding manifest, you can access it in the template by replacing its $ sign with an @. So `$os` becomes `@os`, `$trusted` becomes `@trusted`, etc.

This is the most legible way to access variables, but it doesn’t support variables from other scopes. For that, you need to use the scope object.

### scope['variable'] or scope.lookupvar('variable')

Puppet also passes templates an object called scope, which can access all variables (including out-of-scope ones) with a hash-style access expression. For example, to access `$ntp::tinker` you would use `scope['ntp::tinker']`.


There is also another way to use the `scope` object: you can call its `lookupvar` method and pass the variable’s name as its argument, like `scope.lookupvar('ntp::tinker')`. This is exactly equivalent to the above, but slightly less convenient. It predates the hash-style indexing, which was added in Puppet 3.0.


###Puppet Data Types in Ruby

Puppet’s data types are converted to Ruby classes as follows:

|Puppet type	|Ruby class|
|--|--|--
|Boolean	|Boolean
|Undef	|NilClass (value nil)
|String	|String
|Number	|subtype of Numeric
|Array	|Array
|Hash	|Hash
|Default	|Symbol (value :default)
|Regexp	|Regexp
|Resource reference	|Puppet::Pops::Types::PResourceType, or Puppet::Pops::Types::PHostClassType
|Lambda (code block)	|Puppet::Pops::Evaluator::Closure
|Data type (Type)	|A type class under Puppet::Pops::Types, e.g. Puppet::Pops::Types::PIntegerType

###Testing for Undefined Variables

If a Puppet variable was never defined, its value is `undef`, which means its value in a template will be `nil`.

###Some Basic Ruby for ERB Templates
To manipulate and print data in ERB templates, you’ll need to know a small amount of Ruby. We can’t teach that on this page, but we can give a brief look at some of the things you’re likely to use.

###if statements

Ruby’s `if ... end` statement lets you write conditional text. You’ll want to put the control statements in non-printing tags, and the conditional text between the tags. (Like `<% if <CONDITION> %> text goes here <% end %>`.)

```
<% if @broadcast != "NONE" %>broadcast <%= @broadcast %><% end %>
```

The general format of an if statement is:
```
if <CONDITION>
  ... code ...
elsif <CONDITION>
  ... other code ...
end
```

###Iteration

Ruby lets you iterate over arrays and hashes with the each method. This method takes a block of code, and executes it once for each element in the array or hash. In a template, un-tagged text is treated as part of the code that gets repeated. (It might help to think of literal text as an instruction, telling the evaluator to insert that text into the final output.)

To write a block of code in Ruby, use either `do |arguments| ... end or {|arguments| ... }`. Note that this is different from Puppet’s lambdas — but they work similarly.

```
<% @values.each do |val| -%>
Some stuff with <%= val %>
<% end -%>
```