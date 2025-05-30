---
title: How to write /// docs for .NET API ref
ms.date: 06/29/2023
author: gewarren
ms.author: gewarren
ms.service: learn
ms.topic: contributor-guide
description: Learn how to write good descriptions in source code for generating .NET API reference documentation.
---
# How to write good /// docs for .NET API reference

The ultimate goal for .NET API docs is to have the /// XML comments in the .NET source code be the "source of truth". For MSBuild, ASP.NET Core, and EF Core, this goal has been met. However, currently the [dotnet-api-docs repo](https://github.com/dotnet/dotnet-api-docs) remains the source of truth for core .NET API reference. [This dotnet/runtime issue](https://github.com/dotnet/runtime/issues/44969) tracks the effort to backport .NET docs and make the [dotnet/runtime](https://github.com/dotnet/runtime) repo the source of truth.

This article provides tips about writing good doc comments *within the source code itself*.

## Good comments make for good documents

.NET API triple-slash comments are transformed into public documentation on learn.microsoft.com and also appear in IntelliSense in the IDE. The comments should be:

- Complete&mdash;empty doc entries for methods, parameters, exceptions, and so on, make the APIs feel under-supported, temporary, or trivial.
- Correct&mdash;readers scan for critical details and become frustrated when key information is missing or incorrect.
- Contextual&mdash;readers land on this page from search and need to know how and when to use the API, and what the code implications are.
- Polished&mdash;poor or hasty grammar and spelling can confuse the reader and make even simple calls ambiguous; also, poor presentation communicates low investment.

## Best practices

1. Use `cref` instead of `href` to link to another type or method.

   Correct: `<param name="configFile">An <see cref="XmlConfigResource" /> object.</param>`

   Incorrect:  `<param name="configFile">An <a href="https://learn.Microsoft.com/{path}/XmlConfigResource"></a> object.</param>`

1. When referencing parameters, wrap the parameter name in a `<paramref>` tag, for example, `The offset in <paramref name="source" /> where the range begins.`.
1. If you have more than one paragraph in the doc comment, separate the paragraphs with `<para>` tags.
1. Wrap code examples in `<code>` tags within `<example>` tags.
1. Use `<seealso>` to add links to other APIs in the autogenerated "See Also" section.

## XML doc tags

| Tag | Purpose | Example |
| - | - | - |
| `<altmember>` | Adds a "See also" link to the specified API.  | `<altmember cref="System.Console.Out" />` |
| `<c>` | Formats the specified text as code within a description. | `Gets the current version of the language compiler, in the form <c>Major.Minor.Revision.Build</c>.` |
| `<code>` | Formats multiple lines as code. | `<code language="csharp">using(logger.BeginScope("Processing request from {Address}", address)) { }</code>` |
| `<example>` | Adds a code example under the "Example" H2 heading. | `<example><code language="csharp">using(logger.BeginScope("Processing request from {Address}", address)) { }</code></example>` |
| `<exception>` | Describes an exception the API can throw. | `<exception cref="T:System.ArgumentException">No application identity is specified in <paramref name="identity" />.</exception>` |
| `<include>` | Refer to comments in another file that describe the APIs in your source code. | `<include file="../docs/AbsoluteLayout.xml" path="Type[@FullName='Microsoft.Maui.Controls.AbsoluteLayout']/Docs/*" />`<br/><br/>[.NET MAUI example](https://github.com/dotnet/maui/blob/a064bf8dd9c74909e989fe007053fc05442a7465/src/Controls/src/Core/Layout/AbsoluteLayout.cs#L9) |
| `<inheritdoc>` | Inherit XML comments from base classes, interfaces, and similar methods. | `<inheritdoc />` |
| `<list>` | Creates a bulleted or numbered list. | `<list type="bullet"><item><description>Set the root path to the result.</description></item><item><description>Load host configuration.</description></item></list>` |
| `<para>` | Separates paragraphs. | |
| `<paramref>` | Refers to a method parameter. | `Returns the activity with the specified <paramref name="id" />.` |
| `<related>` | Adds a "See also" link to the specified article. | `<related type="Article" href="/dotnet/framework/ado-net-overview">ADO.NET overview</related>` |
| `<see cref>` | Links to another API. | `Describes the behavior that caused a <see cref="Scroll" /> event.` |
| `<see langword>` | Formats the specified text as code. | `Gets the value of the <see langword="Accept-Ranges" /> header for an HTTP response.` |
| `<seealso>` | Adds a "See also" link to the specified API. | `<seealso cref="T:System.Single" />` |
| `<typeparamref>` | Refers to a type parameter. | `The <typeparamref name="THandler" /> is resolved from a scoped service provider.` |

For more information, see [Recommended XML tags for C#](/dotnet/csharp/language-reference/xmldoc/recommended-tags) and the [C# specification](/dotnet/csharp/language-reference/language-specification/documentation-comments#d2-introduction). The [ECMAXML spec](http://docs.go-mono.com/index.aspx?link=man%3amdoc(5)) also has good information, although be aware that there are some differences between ECMAXML and /// documentation comments (for example, cref targets are fully expanded and have prefixes in ECMAXML).

## Cross references

When you use a `<see cref>` tag to link to another API, there's no need to add a prefix to the type name, such as `T:` for type or `M:` for method. In fact, code analysis [rule CA1200](/dotnet/fundamentals/code-analysis/quality-rules/ca1200) flags code comments that add a prefix to the type name in a `cref` tag. However, there are a couple exceptions to this rule:

- When you want to link to the general form of a method that has more than one overload, the C# compiler [doesn't currently support that](https://github.com/dotnet/csharplang/issues/320). The workaround for docs is to prefix the method name with `O:` in source code (or `Overload:` in ECMAXML) and suppress [rule CA1200](/dotnet/fundamentals/code-analysis/quality-rules/ca1200). For example: `<altmember cref="O:System.Diagnostics.Process.Kill" />`.
- When the API can't be resolved from the current context, which includes any `using` directives. In this case, use the fully qualified API name with a prefix.

When the `<see cref>` tag is converted to [ECMAXML](http://docs.go-mono.com/index.aspx?link=man%3amdoc(5)), mdoc replaces the type name with the full DocId of the API, which includes a prefix.

## Descriptions

For authoritative guidelines about describing each symbol type and its various parts, see the [.NET API docs wiki](https://github.com/dotnet/dotnet-api-docs/wiki).

## Empty comments

The well-known placeholder text for empty comments is `To be added.`. The Learn build system recognizes this text and removes it when the ECMAXML is converted into HTML, leaving an empty description.

## Separate code files

If your code example is lengthy, you can put it in a separate file in the docs repo and link to it from source code in the following way:

```csharp
/// <example>
/// <format type="text/markdown">
/// <![CDATA[
///  [!code-csharp[FieldAware](~/docs/samples/Microsoft.ML.Samples/Dynamic/FactorizationMachine.cs)]
/// ]]></format>
/// </example>
```

For some more details about how to hook up separate code files, see [this discussion](https://github.com/dotnet/runtime/pull/111177#discussion_r1907549328).

## Language attributes

Language attributes on `<code>` tags are optional, but they cause the code to be formatted with color coding. For example:

```csharp
/// <example>
/// This sample shows the basic pattern for defining a typed client class.
///   <code language="csharp">
///     class ExampleClient
///     {
///       private readonly HttpClient _httpClient;
///       private readonly ILogger _logger;
///
///       // Typed clients can use constructor injection to access additional services.
///       public ExampleClient(HttpClient httpClient, ILogger&lt;ExampleClient&gt; logger)
///       {
///         _httpClient = httpClient;
///         _logger = logger;
///       }
///     }
///   </code>
/// </example>
```

## Internal APIs

When documenting an API that's not intended to be used by consumers, use wording similar to the following:

`<summary>This type supports the .NET infrastructure and is not intended to be used directly from your code.</summary>`

## See also

- [ECMAXML format](http://docs.go-mono.com/index.aspx?link=man%3amdoc(5))
- [Recommended XML tags for C#](/dotnet/csharp/language-reference/xmldoc/recommended-tags)
- [Documentation comments (C# specification)](/dotnet/csharp/language-reference/language-specification/documentation-comments#d2-introduction)
