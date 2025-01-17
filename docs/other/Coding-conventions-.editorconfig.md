---
layout: page
title: Dev - Coding conventions with .editorconfig
parent: Other
---

# Coding conventions with .editorconfig

The Microsoft Framework Coding Conventions include Rules for C# to use specific naming for fields, constants, Properties and more. Visual Studio itself lets you code in your own style without forcing you to use a specific formatting or naming style. To have better Code overall I recommend to use [Codemaid](https://marketplace.visualstudio.com/items?itemName=SteveCadwallader.CodeMaid) and a rule set of Coding Conventions, even more so if you don't just work for yourself but in a team. The `.editorconfig` files can help you with that.


## conventions examples and why they are important

I've seen code in real life and productive environments that made me wish, that these projects had an `.editorconfig` file already built in. That's why I researched a way to not only implement rule sets on my own Visual Studio and it's options, but to share theses rules within a file added to the project.

### Visual Studio Options Naming Conventions

[![Visual Studio Options Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/VS_Options_CodeStyle_Naming.png)](/assets/images/articles/coding-confentions-.editorconfig/VS_Options_CodeStyle_Naming.png)

These options are only applied in your own Visual Studio and don't get shared with team members.


### The `.editorconfig` file

This `.editorconfig` file enforces common naming conventions and coding styles for C# projects. It includes rules to ensure consistency and readability across the codebase.


```ini
# Top-most EditorConfig file
root = true

# Apply these settings to all C# files
[*.cs]

# SpellChecking config
spelling_languages = en-us,de-de
spelling_checkable_types = identifiers,comments,strings
spelling_error_severity = hint

# Overall severity level for IDE1006 aka Naming rule violations
#dotnet_diagnostic.IDE1006.severity = warning

# Field names use CamelCase and an underscore prefix
dotnet_naming_rule.field_should_be_camel_case.severity = warning
dotnet_naming_rule.field_should_be_camel_case.symbols = fields
dotnet_naming_rule.field_should_be_camel_case.style = field_style

dotnet_naming_symbols.fields.applicable_kinds = field
dotnet_naming_symbols.fields.applicable_accessibilities = *
dotnet_naming_symbols.fields.required_modifiers =

dotnet_naming_style.field_style.capitalization = camel_case
dotnet_naming_style.field_style.required_prefix = _

# Constant names use PascalCase
dotnet_naming_rule.constant_should_be_pascal_case.severity = warning
dotnet_naming_rule.constant_should_be_pascal_case.symbols = constants
dotnet_naming_rule.constant_should_be_pascal_case.style = pascal_case

dotnet_naming_symbols.constants.applicable_kinds = field
dotnet_naming_symbols.constants.applicable_accessibilities = *
dotnet_naming_symbols.constants.required_modifiers = const

# Property names use PascalCase
dotnet_naming_rule.property_names_should_be_pascal_case.severity = warning
dotnet_naming_rule.property_names_should_be_pascal_case.symbols = properties
dotnet_naming_rule.property_names_should_be_pascal_case.style = pascal_case

dotnet_naming_symbols.properties.applicable_kinds = property
dotnet_naming_symbols.properties.applicable_accessibilities = *
dotnet_naming_symbols.properties.required_modifiers = 

# Method names use PascalCase
dotnet_naming_rule.methods_should_be_pascal_case.severity = warning
dotnet_naming_rule.methods_should_be_pascal_case.symbols = methods
dotnet_naming_rule.methods_should_be_pascal_case.style = pascal_case

dotnet_naming_symbols.methods.applicable_kinds = method
dotnet_naming_symbols.methods.applicable_accessibilities = *
dotnet_naming_symbols.methods.required_modifiers = 

# Parameter names use cascalCase
dotnet_naming_rule.method_parameters_should_be_camel_case.severity = warning
dotnet_naming_rule.method_parameters_should_be_camel_case.symbols = method_parameters
dotnet_naming_rule.method_parameters_should_be_camel_case.style = camel_case

dotnet_naming_symbols.method_parameters.applicable_kinds = parameter
dotnet_naming_symbols.method_parameters.applicable_accessibilities = *
dotnet_naming_symbols.method_parameters.required_modifiers = 

# Variable names use camelCase
dotnet_naming_rule.variable_naming.severity = warning
dotnet_naming_rule.variable_naming.symbols = variable_symbols
dotnet_naming_rule.variable_naming.style = camel_case

dotnet_naming_symbols.variable_symbols.applicable_kinds = local
dotnet_naming_symbols.variable_symbols.applicable_accessibilities = *
dotnet_naming_symbols.variable_symbols.required_modifiers =

# Interface names use PascalCase an I prefix
dotnet_naming_rule.interfaces_should_start_with_i.severity = warning
dotnet_naming_rule.interfaces_should_start_with_i.symbols = interfaces
dotnet_naming_rule.interfaces_should_start_with_i.style = prefix_i

dotnet_naming_symbols.interfaces.applicable_kinds = interface
dotnet_naming_symbols.interfaces.applicable_accessibilities = *
dotnet_naming_symbols.interfaces.required_modifiers = 

dotnet_naming_style.prefix_i.capitalization = pascal_case
dotnet_naming_style.prefix_i.required_prefix = I

# Class names use PascalCase
dotnet_naming_rule.class_names_should_be_pascal_case.severity = warning
dotnet_naming_rule.class_names_should_be_pascal_case.symbols = classes
dotnet_naming_rule.class_names_should_be_pascal_case.style = pascal_case

dotnet_naming_symbols.classes.applicable_kinds = class
dotnet_naming_symbols.classes.applicable_accessibilities = *

# Enum names use PascalCase
dotnet_naming_rule.enum_names_should_be_pascal_case.severity = warning
dotnet_naming_rule.enum_names_should_be_pascal_case.symbols = enums
dotnet_naming_rule.enum_names_should_be_pascal_case.style = pascal_case

dotnet_naming_symbols.enums.applicable_kinds = enum
dotnet_naming_symbols.enums.applicable_accessibilities = *

# pascal_case naming style
dotnet_naming_style.pascal_case.capitalization = pascal_case

# camel_case naming style
dotnet_naming_style.camel_case.capitalization = camel_case
```

#### Summary

**Root Configuration**:
`root = true`: This is the top-most EditorConfig file.

**C# Files**:
`[*.cs]` applies settings to all C# files.

**Naming Conventions**:
- **Fields**:
    - Use camelCase with an underscore prefix.
    - Severity: Warning.
- **Constants**:
    - Use PascalCase.
    - Severity: Warning.
- **Properties**:
    - Use PascalCase.
    - Severity: Warning.
- **Methods**:
    - Use PascalCase.
    - Severity: Warning.
- **Method Parameters**:
    - Use camelCase.
    - Severity: Warning.
- **local variables**:
    - Use camelCase.
    - Severity: Warning.
- **Interfaces**:
    - Use PascalCase with an "I" prefix.
    - Severity: Warning.
- **Classes**:
    - Use PascalCase.
    - Severity: Warning.
- **Enums**:
    - Use PascalCase.
    - Severity: Warning.

This configuration ensures that your C# code follows common naming conventions and coding styles, improving readability and maintainability.


## Examples

### Methods

The method name rule works for methods with all access levels and modifiers (e.g. 'static'):

[![Method Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_method_lower_case_warning.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_method_lower_case_warning.png)

But for methods with a Interface in front of them, the warning is only displayed for the interface definition:

[![Method Interface Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_method_lower_case_interface.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_method_lower_case_interface.png)


### Method parameters

Method parameters should not be named in PascalCase or with leading underscores:

[![Method parameter Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_method_parameter_warning.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_method_parameter_warning.png)


### Local variables

Local variables should be named in camelCase without underscores. Unfortunately you can't set a minimum count for characters.

[![Local variables Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_variables_warning.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_variables_warning.png)


### Constants, Fields and Properties

The rules create warnings for **constants** not being written wit PascalCase or with an unexpected underscore prefix. **Fields** should have an underscore prefix and should be written in camelCase. **Properties** are then PascalCase again.

[![Constants, Fields and Properties Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_constants_fields_properties_warning.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_constants_fields_properties_warning.png)


### Interfaces

Interfaces should have a `I` as prefix and the rest should be written as PascalCase:

[![Interface Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_interface_warning.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_interface_warning.png)


### Classes and Enums

Classes and Enums should be named in PascalCase and even nested definitions get a warning:

[![Classes and Enums Naming Conventions](/assets/images/articles/coding-confentions-.editorconfig/Example_classes_enums_warning.png)](/assets/images/articles/coding-confentions-.editorconfig/Example_classes_enums_warning.png)



## Sources

* [editorConfig Project site](https://editorconfig.org/)
* [Microsoft Coding style rules](https://learn.microsoft.com/de-de/dotnet/fundamentals/code-analysis/style-rules/naming-rules)
* [Microsoft code formatting settings](https://learn.microsoft.com/de-de/visualstudio/ide/code-styles-and-code-cleanup?view=vs-2022)
