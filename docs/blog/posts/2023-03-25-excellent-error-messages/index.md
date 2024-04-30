---
title: "Excellent error messages"
date: 2023-03-25
categories:
  - "programming"
---

Error messages are pervasive throughout programming, yet little has been written on the design of error messages for languages, libraries, and APIs. Much good advice can be found via simple web search on good error messages as shown to end users in GUIs, but standards for error messages intended for an audience of programmers is hard to find. This is not due to a lack of attention to error messages. There are certainly places where error messages are neglected, but neglect is far from universal. In fact, some of the best discussions on good error messages come from specific efforts by big projects to improve their error messages.

<!-- more -->

That error messages are meant to be read by humans instead of computers is probably the main explanation for the absence of standardization. Inconsistent error messages even within a project does not create extra work in the way that an inconsistent API design does. An error message works the same whether it says "Division by zero" or "Divide by zero" in a way that "object.to_string" does not work the same as "object.string". But this functional insensitivity can be an advantage: error messages can be updated without breaking backwards compatibility.

There are 6 main features of an excellent error message:

- ID

- Location

- Context

- Expected

- Actual

- Suggestions (optional)

## ID

This is a stable and unique ID for errors of this type. Typical IDs would be `IndexOutOfBounds` or `NotFound`. I would recommend that these are automatically generated from the exception class in any language whenever that is appropriate. In some sense, this is the least important part of the error message. It should not provide any information to the user that is not spelled out more clearly later in the message. However, it is relatively low-cost to add and has some useful properties.

The ID functions as a title for the error message. A programmer experienced with the language, library, or API could recognize the error at the first word and not even need to read the rest of the message.

The ID also functions as something that is searchable on the web or on documentation. The human-readable text may not be amenable to search or the human-readable text may be improved in subsequent versions, which would otherwise make it hard to find still-valid documentation written against previous versions.

Finally, the ID is a stable, machine-readable part of the error. Sometimes, it is appropriate for machines to read errors, particularly when it is possible for them to automatically recover from them. This is prevalent in languages that use exceptions for flow control (a bad idea that deserves its own blog post), but this is also necessary to use various APIs. For example, an endpoint receiving an OAuth2 access token may return the [following error message](https://www.oauth.com/oauth2-servers/making-authenticated-requests/refreshing-an-access-token/):

```json
{
  "error": "invalid_token",
  "error_description": "The access token expired"
}
```

The `invalid_token` is an ID tells OAuth2 clients that they should try to refresh the access token. Ideally, the application refreshes the token and retries the request without the user being involved or even informed.

## Location

The location of the error is relevant when the error is being generated from inside a complex system. The stack trace is the most famous incarnation of this, but the line number of a compiler error and the location in a JSON data structure serve this function also. This is probably the most important item on the list. I would rather get a "Something went wrong on line 54 of file X" than a "Segmentation Fault" with no location information. Now, an error that simply says "something went wrong" is particularly egregious, but it is better to get that and the location rather than the specific error and no location.

## Context

The context of an error is anything "nearby" the error that is not directly related to the error. Compilers typically do a good job of this by showing the text where the error occurred. Parsita, for example, produces errors like this:

```python
# parsita.state.ParseError: Expected positive integer but found '0'
# Line 10, character 17
#
# 'n_replicates': 0,
#                 ^
```

Printing the entire line is context. The caret pointer does not actually contain any additional information; it is just "Line 10, character 17" in graphical form, but marrying the location to the context can be convenient to the user.

In an API that takes a large object as input, context is the particular piece that failed validation. For example, an API that took a list of model names to simulate would be painful to debug if the error message was just "Model not found" rather than this:

```json
{
    "location": ["models", 5],
    "type": "model_not_found",
    "message": "Model 'mouse_pk' not found"
}
```

## Expected and actual

If most projects were to list out the important attributes of an error message, I would suspect that they would not mention "expected" and "actual" and, instead, say something about "message" or "description". In fact, this is the main motivation of this post. What was expected and what was actually received is important information that is often missing from error messages today. Take the `IndexError` from Python 3.10:

```python
# test.py
def main():
  a = [1,2,3]
  i = 3
  a[i]

main()

# python test.py
# Traceback (most recent call last):
#   File "/home/david/test.py", line 6, in <module>
#     main()
#   File "/home/david/test.py", line 4, in main
#     a[i]
# IndexError: list index out of range
```

This error has a great ID, passable location information, and decent context. What is wrong with it is the message "list index out of range". This is a very common message format—fact about what went wrong—and I hate it. What is the legal range? What was the illegal value? No idea; power up your debugger.

A better error message would be:

```python
# IndexError
# Expected: Integer between -3 and 2
# Actual: 3
```

I am unsure if this multiline style should be the standard. It is definitely different from what people are used to. A more standard message that conveys the same information would be "Expected an integer between -3 and 2, but received 3". A standard style would make it easier to read error messages in the same way that having an error ID makes them easier to read. Interestingly, providing the wrong type to an indexing operation in Python does follow the expected/actual pattern.

```python
[1,2,3]['a']
# SyntaxWarning: list indices must be integers or slices, not str; perhaps you missed a comma?
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# TypeError: list indices must be integers or slices, not str
```

So this is something that Python does sometimes, but not always. I suspect that these message strings are assembled eagerly and Python wants to return a static string in the `IndexError` case because formatting a string would slow down the code when using `IndexError` is caught for control flow, whereas catching a `TypeError` is rarely on the critical path. Python would probably do better to modify the `IndexError` class to have a two attributes `length` and `index` and then have `__str__` generate a good error message lazily. (I leave as an exercise to the reader where to put the comma to make the above code a legal Python statement.)

Rust, which does not use index out of bounds for flow control, does include this information in its error message, albeit not in a standardized format:

```rust
fn main() {
    vec![1, 2, 3][3];
}
// thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 3', src/main.rs:2:5
```

## Suggestions

Suggestions in error messages are the cherry on top of an already good error message. They are genuinely optional because a good enough language with good enough error messages about what went wrong should not need suggestions. The user should be able to figure out what to do without them. And suggestions are not without risk. They can actively make the error messages worse. For example, the Python index error above suggesting that a comma was missing is simply ridiculous.

If the suggestion was always correct, then the language is actually redundant. It could simply apply that suggestion for the user. But in most cases, there is simply nothing to suggest. There is a set of expected values and what was provided as not one of them.

There are two situations where suggestions shine. The first is when there is ambiguity in what the user intended and some action is required by the user to resolve that ambiguity. For example, when interpreting a boolean NumPy array as a Python boolean, the operation is ambiguous when there is more than one element. NumPy raises an error with a helpful suggestion

```python
if np.array([1,2,3]) == 1:
  pass
# ValueError: The truth value of an array with more than one
# element is ambiguous. Use a.any() or a.all()
```

Now, using `any` or `all` may still not be what was intended. In my experience, it always indicates an error elsewhere, like a failure to vectorize this section of the code. Even good suggestions run the risk of being counterproductive.

The second situation where suggestions shine is where there is only one interpretation of the user's request, but it is not safe, so the language or API requires an extra hurdle to make extra sure that that is what the user intended.

## Standardization

This post is mainly a plea for more information to be attached to the exception, mainly in terms of what was expected, what was received, and if part of a larger input, the immediate context of the error. I am unsure about how much there is to gain from standardization here. A standardized message of the form "Expected: blah \\n Actual: blah" makes it quick to read, but not all errors fit so neatly into this form. "Expected: object to be in database; Actual: object was not in database; ID: foo" contains no more information than "Object foo not found".

In a language like Python, it is likely to be cleanest to make a custom exception class for each exception type, use attributes to store the relevant context, and implement the `__str__` method to actually compose the message. Python already does well with the ID, location, and context. Including the expected and actual input is in the hands of the programmer.
