## Scanner
The sourcecode for the TypeScript scanner is located entirely in `scanner.ts`. Scanner is *controlled* internally by the `Parser` to convert the source code to an AST. Here is what the desired outcome is.

```
SourceCode ~~ scanner ~~> Token Stream ~~ parser ~~> AST
```

### Usage by Parser
There is a *singleton* `scanner` created in `parser.ts` to avoid the cost of creating scanners over and over again. This scanner is then *primed* by the parser on demand using the `initializeState` function.

Here is a *simplied* version of the actual code in the parser that you can run demonstrating this concept:

```ts
import {createScanner, syntaxKindToName, SyntaxKind} from "ntypescript";

// TypeScript has a singelton scanner
const scanner = createScanner(ts.ScriptTarget.Latest, /*skipTrivia*/ true);

// That is initialized using a function `initializeState` similar to
function initializeState(text: string) {
    scanner.setText(text);
    scanner.setOnError((message: ts.DiagnosticMessage, length: number) => {
        console.error(message);
    });
    scanner.setScriptTarget(ts.ScriptTarget.ES5);
    scanner.setLanguageVariant(ts.LanguageVariant.Standard);
}

// Sample usage
initializeState(`
var foo = 123;
`);

// Start the scanning
var token = scanner.scan();
while (token != SyntaxKind.EndOfFileToken) {
    console.log(syntaxKindToName(token));
    token = scanner.scan();
}
```

This will print out the following :

```
VarKeyword
Identifier
FirstAssignment
FirstLiteralToken
SemicolonToken
```

### Scanner State
After you call `scan` the scanner updates its local state (position in the scan, current token details etc). The scanner provides a bunch of utility functions to get the current scanner state.

// TODO: document a few of these and provide a sample.

### Standalone scanner
Even though the parser has a singleton scanner you can create a standalone scanner using `createScanner` and use its `setText`/`setTextPos` to scan at different points in a file for your amusement.