# ast-grep VSCode: A Powerful Tool for Structural Search and Replace

Hi, I'm [Herrington](https://twitter.com/hd_nvim).

If you've ever searched code with regular expressions, you may have struggled with matching multiple lines, nested structures, or ignoring comments.

Let me introduce ast-grep VSCode, a new extension leveraging the power of structural search and replace (SSR) to help you perform more precise and efficient search and replace.


## The Limitations of Textual Search and Replace

Suppose you want to refactor your JavaScript code to replace the lodash `_.filter` function with the native `Array.prototype.filter` method. A simple text search and replace might look like this:

```regex
// Find
_.filter\((\.+), (.+)\)
// Replace
$1.filter($2)
```

This might work for some cases, but it has several limitations:

- It can only match single-line expressions. If your code spans multiple lines, you'll miss some matches or get incorrect replacements.
- It can't handle nested structures like parentheses, brackets, or braces. If your code has complex expressions, you'll get incorrect matches or replacements.
- It can't ignore comments or other irrelevant parts of the code. If your code has comments that contain the search pattern, you'll get unwanted matches or replacements.
- It requires you to use match groups or named capture groups to preserve the arguments of the function. This can be tedious and error-prone, especially with many arguments or nested functions.

## Structural Search and Replace Comes to the Rescue

Can we make our search algorithm smarter, understand our code better, and be easier to use? Yes! Structural search and replace (SSR) comes to the rescue!

Structural search and replace (SSR) is a technique that allows you to find and modify code patterns based on their syntax and semantics, not just their text. Instead of treating code as plain text, SSR treats code as a collection of nodes with types, properties, and relationships.

ast-grep is a command-line tool that implements SSR. The query for the lodash example above is quite simple.

```javascript
// Search pattern:
_.filter($ARR, $FUNC)
// Rewrite:
$ARR.filter($FUNC)
```

The pattern illustrates the basic usage of ast-grep. `$ARR` and `$FUNC` are meta-variables. Meta-variables are like the dot `.` in regular expressions, except they match AST nodes instead of characters. You can also use captured meta-variables in the rewrite pattern.

Other than the meta-variables, I hope the pattern and rewrite are self-explanatory.

This query has several advantages over the text search and replace:

- It can match expressions across multiple lines, as long as they are syntactically valid.
- It can handle nested structures like parentheses, brackets, or braces, as they are part of the AST.
- It can ignore comments or other irrelevant parts of the code, as they are not part of the AST.
- It does not require match groups or named capture groups, as the meta-variables are accessed by their names in the search pattern.


### Unique Features of ast-grep

ast-grep uses the tree-sitter library, a fast and robust parser for many programming languages. ast-grep allows you to write queries using a simple pattern syntax that resembles code and apply them to files or directories of code. Some of the unique features of ast-grep are:

- It supports many languages, including JavaScript, TypeScript, Python, Ruby, Java, C#, Go, Rust, and more. You can also add support for new languages by registering tree-sitter grammars in the configuration.
- It is written in Rust, which makes it very fast and memory-efficient. It can process large codebases in a matter of seconds.
- It has a rich set of options and flags, such as recursive search, dry run, interactive mode, color output, and more. You can customize your SSR experience to suit your needs and preferences.

But, ast-grep until now only has a command-line interface. Can we have its power right near our hands instead of switching between terminals and editors?

## ast-grep VSCode: Bridging the CLI and the Editor

ast-grep VSCode is a new extension that integrates ast-grep with Visual Studio Code, one of the most popular code editors. With ast-grep VSCode, you can use SSR within your editor, without leaving your workflow.

### Structural Search, Replace, and More

Some features of ast-grep VSCode include:

- It provides a user interface for writing and executing SSR queries, and you can see the results of your queries in a sidebar, with previews and diffs.
- It also supports linting and code fix. You can set up an ast-grep project and write custom rules tailored to your needs.
- It feels like a native VSCode feature, with seamless integration and consistent design.

| Feature         | Screenshot                                                                                                  |
| --------------- | ----------------------------------------------------------------------------------------------------------- |
| Search Pattern  | <img src="https://github.com/ast-grep/ast-grep-vscode/blob/main/readme/search-pattern.png?raw=true">     |
| Search In Folder| <img src="https://github.com/ast-grep/ast-grep-vscode/blob/main/readme/search-in-folder.png?raw=true">  |
| Replace Preview | <img src="https://github.com/ast-grep/ast-grep-vscode/blob/main/readme/replace.png?raw=true">             |
| Commit Replace  | <img src="https://github.com/ast-grep/ast-grep-vscode/blob/main/readme/commit-replace.png?raw=true">     |
| Code Linting    | <img src="https://github.com/ast-grep/ast-grep-vscode/blob/main/readme/linter.png?raw=true">               |

## Comparing with Other SSR Tools

There are other SSR tools and extensions available. However, I believe ast-grep VSCode is one of the best SSR extensions for VSCode because:

- It has good performance, backed by a multi-threaded CLI written in a native language.
- It supports multiple languages, leveraging the tree-sitter grammars widely used and maintained by the community.
- It has a user-friendly and intuitive interface that feels like a VSCode built-in feature.

Finally, I want to highlight some React techniques used in ast-grep VSCode that make it fast and responsive:

- [`useSyncExternalStore`](https://github.com/ast-grep/ast-grep-vscode/blob/789d27325fb9ce6a0bb969caefdc718942f6d2b3/src/webview/hooks/useSearch.tsx#L150-L159) was used to manage streaming results from ast-grep CLI. This hook allows components to subscribe to an external mutable source of data and update the rendering accordingly. This way, the extension can show the results as soon as they are available.
- [`useDeferredValue`](https://github.com/ast-grep/ast-grep-vscode/blob/main/src/webview/SearchSidebar/index.tsx#L13-L16) deferred rendering the result list. This hook can avoid blocking user input and improve the perceived performance of the UI by delaying the update of a derived state until the next concurrent render.
- The extension carefully used plenty of `memo` and CSS tricks to reduce the JavaScript workload.

## Conclusion

I hope this article helps you understand the benefits of SSR and the features of ast-grep VSCode. If you are interested in trying it out, you can install it from the [VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=ast-grep.ast-grep-vscode). ast-grep VSCode is still in development and has a lot of room for improvement. I would love to hear your feedback and suggestions, so feel free to open an issue or a pull request on [GitHub](https://github.com/ast-grep/ast-grep-vscode).
Happy grepping!

(1) It might not be the best SSR plugin for neovim, though. [ssr.nvim](https://github.com/cshuaimin/ssr.nvim)
