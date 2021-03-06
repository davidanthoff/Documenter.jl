# Doctests

Documenter will, by default, try to run `jldoctest` code blocks that it finds in the generated
documentation. This can help to avoid documentation examples from becoming outdated,
incorrect, or misleading. It's recommended that as many of a package's examples be runnable
by Documenter's doctest.

This section of the manual outlines how to go about enabling doctests for code blocks in
your package's documentation.

## "Script" Examples

The first, of two, types of doctests is the "script" code block. To make Documenter detect
this kind of code block the following format must be used:

````markdown
```jldoctest
a = 1
b = 2
a + b

# output

3
```
````

The code block's "language" must be `jldoctest` and must include a line containing the text `#
output`. The text before this line is the contents of the script which is run. The text that
appears after `# output` is the textual representation that would be shown in the Julia REPL
if the script had been `include`d.

The actual output produced by running the "script" is compared to the expected result and
any difference will result in [`makedocs`](@ref) throwing an error and terminating.

Note that the amount of whitespace appearing above and below the `# output` line is not
significant and can be increased or decreased if desired.

## REPL Examples

The other kind of doctest is a simulated Julia REPL session. The following format is
detected by Documenter as a REPL doctest:

````markdown
```jldoctest
julia> a = 1
1

julia> b = 2;

julia> c = 3;  # comment

julia> a + b + c
6

```
````

As with script doctests, the code block must have it's language set to `jldoctest`. When a code
block contains one or more `julia> ` at the start of a line then it is assumed to be a REPL
doctest. Semi-colons, `;`, at the end of a line works in the same way as in the Julia REPL
and will suppress the output, although the line is still evaluated.

Note that not all features of the REPL are supported such as shell and help modes.

## Exceptions

Doctests can also test for thrown exceptions and their stacktraces. Comparing of the actual
and expected results is done by checking whether the expected result matches the start of
the actual result. Hence, both of the following errors will match the actual result.

````markdown
```jldoctest
julia> div(1, 0)
ERROR: DivideError: integer division error
 in div(::Int64, ::Int64) at ./int.jl:115

julia> div(1, 0)
ERROR: DivideError: integer division error

```
````

If instead the first `div(1, 0)` error was written as

````markdown
```jldoctest
julia> div(1, 0)
ERROR: DivideError: integer division error
 in div(::Int64, ::Int64) at ./int.jl:114

```
````

where line `115` is replaced with `114` then the doctest will fail.

In the second `div(1, 0)`, where no stacktrace is shown, it may appear to the reader that
it is expected that no stacktrace will actually be displayed when they attempt to try to
recreate the error themselves. To indicate to readers that the output result is truncated
and does not display the entire (or any of) the stacktrace you may write `[...]` at the
line where checking should stop, i.e.

````markdown
```jldoctest
julia> div(1, 0)
ERROR: DivideError: integer division error
[...]

```
````

## Skipping Doctests

Doctesting can be disabled by setting the [`makedocs`](@ref) keyword `doctest = false`.
This should only be done when initially laying out the structure of a package's
documentation, after which it's encouraged to always run doctests when building docs.

## Setup Code

Doctests may require some setup code that must be evaluated prior to that of the actual
example, but that should not be displayed in the final documentation. It could also be that
several separate doctests require the same definitions. For both of these cases a `@meta`
block containing a `DocTestSetup = ...` value can be used as follows:

    ```jldoctest
    julia> using DataFrames

    julia> df = DataFrame(A = 1:10, B = 2:2:20);

    ```

    Some text discussing `df`...

    ```@meta
    DocTestSetup = quote
        using DataFrames
        df = DataFrame(A = 1:10, B = 2:2:20)
    end
    ```

    ```jldoctest
    julia> df[1, 1]
    1
    ```

    Some more text...

    ```jldoctest
    julia> df[1, :]
    1x2 DataFrames.DataFrame
    | Row | A | B |
    |-----|---|---|
    | 1   | 1 | 2 |
    ```

    ```@meta
    DocTestSetup = nothing
    ```

Note that the `DocTestSetup` value is **re-evaluated** at the start of *each* doctest block
and no state is shared between any code blocks. The `DocTestSetup = nothing` is not strictly
necessary, but good practice nonetheless to help avoid unintentional definitions later on a
page.
