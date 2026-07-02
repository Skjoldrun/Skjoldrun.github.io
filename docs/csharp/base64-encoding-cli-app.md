---
layout: page
title: C# - Base64 En/Decoding CLI App
parent: C#
---

# Understanding Base64 — and Building a Small, Clean CLI for It in .NET

Base64 is one of those technologies that is *everywhere* — in data URLs, JWT
tokens, email attachments, API payloads — yet many developers only ever
copy-paste it without really knowing what happens under the hood. In this
article we do two things:

1. **Explain Base64 properly**: what it is, why it exists, how it works, what
   padding means, and what makes the "URL-safe" variant different.
2. **Walk through a small, well-structured .NET 10 command-line tool**
   (`EncodingEDU`) that implements Base64 encoding and decoding — a clean,
   readable example of how to build a console application with proper argument
   handling, meaningful exit codes, and a testable design.

---

## Part 1 — What is Base64?

### The core idea

Base64 is a **binary-to-text encoding**. It represents arbitrary binary data
(bytes) using only a small set of printable ASCII characters.

Two things it is **not**:

- It is **not encryption.** Anyone can decode it; it hides nothing.
- It is **not compression.** In fact, the output is *larger* than the input.

So why use it? Because of **safe transport**. Many systems — email headers,
URLs, JSON, XML, HTTP tokens — were designed to carry *text*, not raw bytes.
Feed them a control character or a `0x00` byte and they may corrupt, truncate,
or misinterpret it. Base64 lets binary data travel unharmed through text-only
channels.

### How the encoding works

Base64 processes the input **3 bytes at a time**. Three bytes are 24 bits,
which get re-grouped into **four 6-bit chunks**. Each 6-bit value (0–63) maps to
exactly one character from a 64-character **alphabet**:

```
A–Z  (0–25)   a–z (26–51)   0–9 (52–61)   +  (62)   /  (63)
```

A worked example for the three ASCII bytes `M`, `a`, `n`:

```
Text:      M            a            n
ASCII:     77           97           110
Binary:    01001101     01100001     01101110

Regroup into 6-bit chunks:
           010011  010110  000101  101110
Decimal:   19      22      5       46
Base64:    T       W       F       u

Result:    "TWFu"
```

Three input bytes → four output characters. That 4:3 ratio is exactly why Base64
output is roughly **33% larger** than the original — the price you pay for text
safety.

### Padding: those trailing `=` characters

The input is rarely a neat multiple of 3 bytes. When the final group has only
1 or 2 bytes left over, Base64 still emits a full 4-character block and fills
the remainder with the **padding character `=`**:

| Remaining bytes | Encoded block ends with |
| --------------- | ----------------------- |
| 2 bytes         | 3 chars + `=`           |
| 1 byte          | 2 chars + `==`          |
| 0 (exact fit)   | no padding              |

Padding keeps the encoded length a multiple of 4, so a decoder always knows
where block boundaries are. A useful consequence of this rule: a valid Base64
length is **never** `length % 4 == 1` — such a string is malformed.

Example:

```
"Hi"  -> two bytes  -> "SGk="      (one '=')
"H"   -> one byte   -> "SA=="      (two '=')
"Man" -> three bytes-> "TWFu"      (no padding)
```

### The URL-safe variant

The standard alphabet contains `+` and `/`, and it uses `=` for padding. All
three are **problematic in URLs and file names**:

- `/` is a path separator
- `+` is often interpreted as a space in query strings
- `=` is the key/value separator in query strings

The **URL- and filename-safe** alphabet (defined in RFC 4648 §5) solves this
with two substitutions and a convention:

- `+` becomes `-`
- `/` becomes `_`
- the trailing `=` padding is usually **dropped**

The data is identical — only the *presentation* changes so it can live safely in
a URL or a file name.

### Things to watch out for

- **Decode the same way you encoded.** Standard and URL-safe Base64 are *not*
  interchangeable. A URL-safe string contains `-`/`_` and often lacks padding;
  feeding it to a standard decoder produces an "invalid Base64" error.
- **It is not security.** Never treat an encoded secret as protected. Use real
  encryption for that.
- **Whitespace can invalidate it.** Stray spaces or newlines inside the encoded
  text may make it invalid, so tools often trim surrounding whitespace before
  decoding.
- **Size grows by ~33%.** Keep this in mind when encoding large files.

> **Reference:** Base64 is specified in
> [RFC 4648](https://www.rfc-editor.org/rfc/rfc4648). The standard alphabet is in
> §4; the URL- and filename-safe alphabet is in §5.

---

## Part 2 — EncodingEDU: a clean Base64 CLI in .NET 10

With the theory in place, let's look at a small program that puts it to work.
`EncodingEDU` is an **educational** .NET 10 command-line tool that encodes and
decodes data using Base64. Its goal is not to be feature-packed, but to be a
*clean, easy-to-read* example of a well-structured console application:

- a single-responsibility core service,
- a tiny dependency-free argument parser,
- an immutable options model,
- and meaningful exit codes.

### Project layout

The application lives in one project, `EncodingEDU.UI`, split into small files
by responsibility:

```
EncodingEDU.UI/
├── Program.cs          // Entry point + I/O wiring (the "Cli" host)
├── ArgumentParser.cs   // Turns string[] args into a CliOptions object
├── CliOptions.cs       // Immutable record describing one invocation
├── CommandKind.cs      // enum: Encode | Decode
└── Base64Service.cs    // The pure encoding/decoding logic
```

The design keeps the **pure logic** (`Base64Service`) completely separate from
**argument parsing** (`ArgumentParser`) and from **I/O and process concerns**
(`Program.cs`). That separation is what makes the core easy to unit-test.

### The core: `Base64Service`

This is the heart of the program. It knows nothing about the command line — it
just turns bytes into Base64 text and back, with optional URL-safe handling.
Notice how it builds on the framework's `Convert.ToBase64String` /
`Convert.FromBase64String` and layers the URL-safe transformation on top.

```csharp
namespace EncodingEDU.UI;

/// <summary>
/// Provides Base64 encoding and decoding for raw byte payloads, with optional
/// support for the URL-safe alphabet (RFC 4648 §5).
/// </summary>
internal static class Base64Service
{
    /// <summary>
    /// Encodes the supplied bytes to a Base64 string.
    /// </summary>
    /// <param name="data">The raw bytes to encode.</param>
    /// <param name="urlSafe">When <c>true</c>, uses the URL-safe alphabet without padding.</param>
    public static string Encode(ReadOnlySpan<byte> data, bool urlSafe)
    {
        string encoded = Convert.ToBase64String(data);
        return urlSafe ? ToUrlSafe(encoded) : encoded;
    }

    /// <summary>
    /// Decodes the supplied Base64 text back into raw bytes.
    /// </summary>
    /// <param name="text">The Base64 (standard or URL-safe) text to decode.</param>
    /// <param name="urlSafe">When <c>true</c>, interprets the input as the URL-safe alphabet.</param>
    /// <exception cref="FormatException">Thrown when the input is not valid Base64.</exception>
    public static byte[] Decode(string text, bool urlSafe)
    {
        string normalized = urlSafe ? FromUrlSafe(text) : text;
        return Convert.FromBase64String(normalized);
    }

    private static string ToUrlSafe(string standard) =>
        standard.Replace('+', '-').Replace('/', '_').TrimEnd('=');

    private static string FromUrlSafe(string urlSafe)
    {
        string standard = urlSafe.Replace('-', '+').Replace('_', '/');
        int padding = standard.Length % 4;
        if (padding > 0)
        {
            standard = standard.PadRight(standard.Length + (4 - padding), '=');
        }

        return standard;
    }
}
```

The two private helpers are where the URL-safe theory from Part 1 becomes code:

- **`ToUrlSafe`** — after standard encoding, replace `+`→`-` and `/`→`_`, then
  `TrimEnd('=')` to drop the padding.
- **`FromUrlSafe`** — reverse the substitutions, then **re-add the padding** the
  decoder needs. The length is rounded up to the next multiple of 4 with `=`.
  This is why callers can hand in unpadded URL-safe input and it "just works".

> A subtle detail: if `length % 4 == 1`, the string is malformed (as explained
> in Part 1). `FromUrlSafe` will still pad it, but `Convert.FromBase64String`
> then correctly rejects it — so invalid input fails loudly rather than
> silently producing garbage.

### The options model: `CommandKind` and `CliOptions`

The parsed command line is captured in two small, immutable types. First, the
operation to perform:

```csharp
namespace EncodingEDU.UI;

/// <summary>The operation the tool should perform.</summary>
internal enum CommandKind
{
    Encode,
    Decode,
}
```

> **Naming note:** the `Kind` suffix is the idiomatic .NET choice for an enum of
> variants (compare `DateTimeKind`, `SyntaxKind`). `CommandType` would be weaker
> — `Type` evokes reflection, and `System.Data.CommandType` already exists.

Second, the full description of a single invocation, modelled as an **immutable
`record`** with `init`-only properties:

```csharp
namespace EncodingEDU.UI;

/// <summary>
/// The fully parsed set of options describing a single tool invocation.
/// </summary>
internal sealed record CliOptions
{
    public required CommandKind Command { get; init; }

    /// <summary>Inline input value passed via <c>--text</c>. <c>null</c> when not supplied.</summary>
    public string? Text { get; init; }

    /// <summary>Path to read the input from via <c>--file</c>. <c>null</c> when not supplied.</summary>
    public string? InputFile { get; init; }

    /// <summary>Path to write the result to via <c>--out</c>. <c>null</c> writes to stdout.</summary>
    public string? OutputFile { get; init; }

    /// <summary>Use the URL-safe Base64 alphabet without padding.</summary>
    public bool UrlSafe { get; init; }
}
```

Using a `record` here means an invocation, once parsed, is a **value object**:
it can't be mutated halfway through execution, which removes a whole class of
bugs. `required` on `Command` guarantees the parser can never forget to set it.

### Argument parsing: `ArgumentParser`

A tiny, dependency-free parser turns the raw `string[]` into a `CliOptions`.
Note the co-located `CliParseException` at the top — it is only ever thrown by
this parser, so keeping it in the same file next to its only user is a
deliberate, defensible choice for such a small, tightly-coupled type.

```csharp
namespace EncodingEDU.UI;

/// <summary>Raised when the command line arguments cannot be parsed.</summary>
internal sealed class CliParseException(string message) : Exception(message);

/// <summary>
/// A small, dependency-free command line parser for the tool.
/// Grammar: <c>&lt;encode|decode&gt; [--text &lt;value&gt;] [--file &lt;path&gt;] [--out &lt;path&gt;] [--url-safe]</c>.
/// </summary>
internal static class ArgumentParser
{
    /// <summary>
    /// Parses the raw process arguments into a <see cref="CliOptions"/> instance.
    /// </summary>
    /// <exception cref="CliParseException">Thrown when arguments are missing or invalid.</exception>
    public static CliOptions Parse(IReadOnlyList<string> args)
    {
        if (args.Count == 0)
        {
            throw new CliParseException("No command specified.");
        }

        CommandKind command = args[0].ToLowerInvariant() switch
        {
            "encode" or "enc" or "e" => CommandKind.Encode,
            "decode" or "dec" or "d" => CommandKind.Decode,
            var other => throw new CliParseException($"Unknown command '{other}'. Expected 'encode' or 'decode'."),
        };

        string? text = null;
        string? inputFile = null;
        string? outputFile = null;
        bool urlSafe = false;

        for (int i = 1; i < args.Count; i++)
        {
            string arg = args[i];
            switch (arg)
            {
                case "--text" or "-t":
                    text = RequireValue(args, ref i, arg);
                    break;
                case "--file" or "-f":
                    inputFile = RequireValue(args, ref i, arg);
                    break;
                case "--out" or "-o":
                    outputFile = RequireValue(args, ref i, arg);
                    break;
                case "--url-safe" or "-u":
                    urlSafe = true;
                    break;
                default:
                    throw new CliParseException($"Unknown option '{arg}'.");
            }
        }

        if (text is not null && inputFile is not null)
        {
            throw new CliParseException("Use either --text or --file, not both.");
        }

        return new CliOptions
        {
            Command = command,
            Text = text,
            InputFile = inputFile,
            OutputFile = outputFile,
            UrlSafe = urlSafe,
        };
    }

    private static string RequireValue(IReadOnlyList<string> args, ref int index, string option)
    {
        if (index + 1 >= args.Count)
        {
            throw new CliParseException($"Option '{option}' requires a value.");
        }

        return args[++index];
    }
}
```

Highlights worth calling out:

- **The first argument is the command** (`encode`/`decode`, with short aliases).
  Everything after it is an option.
- **`switch` expressions with `or` patterns** keep the alias handling compact
  and readable.
- **Validation lives here**, not in the core: e.g. you can't pass both `--text`
  and `--file`. The parser fails fast with a clear message.
- The parser is **pure** — it reads no files and writes no output; it only
  transforms arguments into data.

### The entry point: `Program.cs`

Finally, the host ties everything together. It uses **top-level statements** for
the entry point and delegates to a small `Cli` class. This layer owns everything
"impure": reading stdin/files, writing stdout/files, catching exceptions, and
mapping them to process exit codes.

```csharp
using System.Reflection;
using System.Text;
using EncodingEDU.UI;

return Cli.Run(args);

internal static class Cli
{
    private const string ExecutableName = "encoding-edu";

    public static int Run(string[] args)
    {
        if (args.Length == 0 || IsHelpRequested(args))
        {
            PrintUsage();
            return args.Length == 0 ? ExitCodes.UsageError : ExitCodes.Success;
        }

        if (IsVersionRequested(args))
        {
            Console.WriteLine(GetInformationalVersion());
            return ExitCodes.Success;
        }

        CliOptions options;
        try
        {
            options = ArgumentParser.Parse(args);
        }
        catch (CliParseException ex)
        {
            Console.Error.WriteLine($"Error: {ex.Message}");
            Console.Error.WriteLine($"Run '{ExecutableName} --help' for usage.");
            return ExitCodes.UsageError;
        }

        try
        {
            Execute(options);
            return ExitCodes.Success;
        }
        catch (Exception ex) when (ex is FormatException or IOException or UnauthorizedAccessException)
        {
            Console.Error.WriteLine($"Error: {ex.Message}");
            return ExitCodes.RuntimeError;
        }
    }

    private static void Execute(CliOptions options)
    {
        if (options.Command == CommandKind.Encode)
        {
            byte[] input = ReadInputBytes(options);
            string result = Base64Service.Encode(input, options.UrlSafe);
            WriteText(options.OutputFile, result);
        }
        else
        {
            string input = ReadInputText(options).Trim();
            byte[] result = Base64Service.Decode(input, options.UrlSafe);
            WriteBytes(options.OutputFile, result);
        }
    }

    private static byte[] ReadInputBytes(CliOptions options)
    {
        if (options.Text is not null)
        {
            return Encoding.UTF8.GetBytes(options.Text);
        }

        if (options.InputFile is not null)
        {
            return File.ReadAllBytes(options.InputFile);
        }

        using var stdin = Console.OpenStandardInput();
        using var buffer = new MemoryStream();
        stdin.CopyTo(buffer);
        return buffer.ToArray();
    }

    private static string ReadInputText(CliOptions options)
    {
        if (options.Text is not null)
        {
            return options.Text;
        }

        if (options.InputFile is not null)
        {
            return File.ReadAllText(options.InputFile);
        }

        return Console.In.ReadToEnd();
    }

    private static void WriteText(string? outputFile, string content)
    {
        if (outputFile is null)
        {
            Console.Out.WriteLine(content);
        }
        else
        {
            File.WriteAllText(outputFile, content);
        }
    }

    private static void WriteBytes(string? outputFile, byte[] content)
    {
        if (outputFile is null)
        {
            using var stdout = Console.OpenStandardOutput();
            stdout.Write(content, 0, content.Length);
        }
        else
        {
            File.WriteAllBytes(outputFile, content);
        }
    }

    private static bool IsHelpRequested(string[] args) =>
        args.Any(a => a is "--help" or "-h" or "-?" or "help");

    private static bool IsVersionRequested(string[] args) =>
        args.Any(a => a is "--version" or "-v");

    private static string GetInformationalVersion()
    {
        Assembly assembly = Assembly.GetExecutingAssembly();
        string? informational = assembly
            .GetCustomAttribute<AssemblyInformationalVersionAttribute>()?
            .InformationalVersion;

        return informational ?? assembly.GetName().Version?.ToString() ?? "unknown";
    }

    private static void PrintUsage()
    {
        Console.WriteLine(
            $"""
            EncodingEDU {GetInformationalVersion()}
            A small educational CLI for Base64 encoding and decoding.

            Usage:
              {ExecutableName} <command> [options]

            Commands:
              encode, enc, e   Encode the input to Base64.
              decode, dec, d   Decode the Base64 input back to its bytes.

            Options:
              -t, --text <value>   Use the given string as input.
              -f, --file <path>    Read the input from a file.
              -o, --out <path>     Write the result to a file (default: stdout).
              -u, --url-safe       Use the URL-safe Base64 alphabet (no padding).
              -h, --help           Show this help text.
              -v, --version        Show the tool version.

            If neither --text nor --file is given, the input is read from stdin.

            Examples:
              {ExecutableName} encode --text "Hello, world!"
              {ExecutableName} decode --text "SGVsbG8sIHdvcmxkIQ=="
              {ExecutableName} encode --file image.png --url-safe --out image.b64
              echo "Hello" | {ExecutableName} encode
            """);
    }
}

internal static class ExitCodes
{
    public const int Success = 0;
    public const int UsageError = 1;
    public const int RuntimeError = 2;
}
```

A few design decisions stand out:

- **Direction of data matters.** Encoding *reads bytes* and *writes text*;
  decoding *reads text* and *writes bytes*. The `Execute` method mirrors this
  asymmetry precisely (`ReadInputBytes` + `WriteText` vs. `ReadInputText` +
  `WriteBytes`).
- **Three input sources, one abstraction.** `--text`, `--file`, and stdin are
  all funnelled into the same code path, so piping "just works".
- **Exception filters map errors to exit codes.** A `when (ex is FormatException
  or IOException or ...)` filter turns *expected* runtime failures (bad Base64,
  missing file, permissions) into a clean error message and exit code `2`,
  instead of a stack-trace dump.
- **`--version` is provided by build tooling.** The version string comes from an
  `AssemblyInformationalVersionAttribute`, which in the real project is stamped
  in at build time by Nerdbank.GitVersioning (derived from the Git history).

### Exit codes

The tool follows the Unix convention of a meaningful process exit code:

| Code | Meaning                                   |
| ---- | ----------------------------------------- |
| `0`  | Success.                                  |
| `1`  | Usage error (bad/missing arguments).      |
| `2`  | Runtime error (invalid Base64, I/O, ...). |

This makes it scriptable: a shell can branch on whether the last invocation
succeeded, failed because the user held it wrong, or failed at runtime.

---

## Part 3 — Using the tool

### Build & run

```powershell
# Build
dotnet build

# Run via the SDK (note the -- separating dotnet args from tool args)
dotnet run --project src/EncodingEDU.UI -- encode --text "Hello, world!"
```

### Commands

| Command              | Description                                |
| -------------------- | ------------------------------------------ |
| `encode`, `enc`, `e` | Encode the input to Base64.                |
| `decode`, `dec`, `d` | Decode the Base64 input back to its bytes. |

### Options

| Option             | Description                                   |
| ------------------ | --------------------------------------------- |
| `-t`, `--text`     | Use the given string as input.                |
| `-f`, `--file`     | Read the input from a file.                   |
| `-o`, `--out`      | Write the result to a file (default: stdout). |
| `-u`, `--url-safe` | Use the URL-safe Base64 alphabet (no padding).|
| `-h`, `--help`     | Show the help text.                           |
| `-v`, `--version`  | Show the tool version.                        |

If neither `--text` nor `--file` is supplied, input is read from **stdin**.

### Examples

```powershell
# Encode a string
encoding-edu encode --text "Hello, world!"
# -> SGVsbG8sIHdvcmxkIQ==

# Decode a string
encoding-edu decode --text "SGVsbG8sIHdvcmxkIQ=="
# -> Hello, world!

# Encode a (binary) file using the URL-safe alphabet into another file
encoding-edu encode --file image.png --url-safe --out image.b64

# Decode it back — remember to keep the --url-safe flag!
encoding-edu decode --file image.b64 --url-safe --out image.png

# Pipe input from stdin
"Hello" | encoding-edu encode
```

> **Reminder from Part 1:** always decode with the same alphabet you encoded
> with. If you produced output with `--url-safe`, you must decode it with
> `--url-safe` too — otherwise the `-`/`_` characters and missing padding will
> trigger an "invalid Base64" error (exit code `2`).

---

## Design takeaways

If you're building your own small CLI, `EncodingEDU` illustrates a handful of
patterns worth stealing:

- **Separate pure logic from I/O.** `Base64Service` has no console, no files, no
  `Console.WriteLine`. That makes it trivial to unit-test in isolation.
- **Model your parsed input as an immutable value.** A `record` with
  `init`-only, `required` properties turns "the parsed command line" into a
  single, tamper-proof object.
- **Fail fast with clear messages, and map failures to exit codes.** Users (and
  scripts) get actionable feedback instead of stack traces.
- **Lean on the framework.** `Convert.ToBase64String` /
  `Convert.FromBase64String` do the heavy lifting; the custom code only adds the
  thin URL-safe layer on top.
- **Keep tiny, tightly-coupled types close to their user.** A one-line
  `CliParseException` next to the parser that throws it is clearer than a
  ceremony-filled file of its own.

Base64 is simple once you see the 3-bytes-to-4-characters rhythm behind it — and
a good CLI is simple once you separate the *what* (encode/decode bytes) from the
*how* (parse args, read input, write output, report errors). Put the two
together and you get a tool that is small, correct, and genuinely easy to read.

---

*Base64 is specified in [RFC 4648](https://www.rfc-editor.org/rfc/rfc4648).*
