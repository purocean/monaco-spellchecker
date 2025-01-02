# monaco-spellchecker

This library provides a simple approach to spellchecking words in the Monaco Editor. [Try it out!](https://purocean.github.io/monaco-spellchecker/)

## Installation

```bash
npm install monaco-spellchecker
```

## Dictionary Library

Please install the [typo-js](https://github.com/cfinke/Typo.js) library if you want to use it as a dictionary for spellchecking.
Or use [hunspell-asm](https://github.com/kwonoj/hunspell-asm) + Web Worker for performance. [For instance, see here](https://github.com/purocean/yank-note-extension/tree/main/packages/extension-spellchecker)

## Example

Below is an example of how to create a dictionary and use it with monaco-spellchecker:

```typescript
import * as monaco from 'monaco-editor'
import Typo from 'typo-js'
import { getSpellchecker } from 'monaco-spellchecker'
import affData from 'typo-js/dictionaries/en_US/en_US.aff?raw'
import wordsData from 'typo-js/dictionaries/en_US/en_US.dic?raw'

const editor = monaco.editor.create(/* ...existing code... */)

// Create dictionary
const dictionary = new Typo("en_US", affData, wordsData)

// Get Spell Checker
const spellchecker = getSpellchecker(monaco, editor, {
  check: (word) => dictionary.check(word),
  suggest: (word) => dictionary.suggest(word),
  ignore: (word) => {
    console.log('Ignore word:', word)
    // ...existing code...
  },
  addWord: (word) => {
    console.log('Add word:', word)
    // ...existing code...
  }
})

const process = debounce(spellchecker.process, 500)

// Process the editor content after it has been changed
editor.onDidChangeModelContent(() => {
  process()
})

// Register code action provider
monaco.languages.registerCodeActionProvider('markdown', spellchecker.codeActionProvider)
```

## API Reference

### getSpellchecker(monaco, editor, options)

Parameters:
- `monaco`: The Monaco instance.
- `editor`: The Editor instance.
- `options`: An object with the following properties:
    | Property           | Type                                                                 | Description                                                                                       |
    |--------------------|----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
    | `check`            | `(word: string) => (boolean \| Promise<boolean>)`                     | Function to check if a word is spelled correctly.                                                 |
    | `suggest`          | `(word: string) => (string[] \| Promise<string[]>)`                   | Function to provide suggestions for a misspelled word.                                            |
    | `languageSelector` | `Monaco.languages.LanguageSelector`                                  | Optional Monaco language selector. Default is `*`.                                                |
    | `severity`         | `Monaco.MarkerSeverity`                                              | Optional severity level for the diagnostic. Default is `monaco.MarkerSeverity.Warning`.           |
    | `tokenize`         | `(text: string) => { word: string, pos: number }[] \| Iterable<{ word: string, pos: number }>` | Optional function to tokenize the text. If not provided, the default behavior is to match `/\b[a-zA-Z']+\b/g`. |
    | `ignore`           | `(word: string) => (void \| Promise<void>)`                          | Optional function to ignore a word. If not provided, the default behavior is to hide the ignore button. |
    | `addWord`          | `(word: string) => (void \| Promise<void>)`                          | Optional function to add a word to the dictionary. If not provided, the default behavior is to hide the add button. |
    | `messageBuilder`   | `(type: 'hover-message' \| 'ignore' \| 'add-word' \| 'apply-suggestion', word: string, range?: XRange, opts?: Options) => string` | Optional function to build message.                                                               |

Returns an object with:
1. `process()`: Re-scans the editor content for misspelled words.
2. `dispose()`: Disposes the spellchecker.
