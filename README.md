# code-to-essay
The library essay is a code generator from a readme file but this library is a reverse operation.

## Inspiration
I used `essay` and made a simple project that can generate many code files of javascript (ES7). That can create a good document because you can update code and docs in the same time. I would like to create a converter from code files structure into readme. It will be also created by `essay` after convert them.

## Getting started
  1. Install this library

  ```
  npm i -g code-to-essay
  ```

  2. Write a pathname with prefix `c2e:` in your README.md

  ```
  c2e:src/main.js
  ```

  3. Convert them with command
  ```
  c2e
  ```

  4. Setting your project follow `essay` format ([https://github.com/dtinth/essay](https://github.com/dtinth/essay))


*Now, the project could be generated by `essay`.*


## Implementation

`main.js` will start first when you call `c2e build`.

```js
// main.js
#!/usr/bin/env node
import { getTextFromFile, writeFileFromText } from './file'
import { convertLinesToContexts } from './converter'
const README = getTextFromFile(`./README.md`)
const lines = README.split('\n')
const newContexts = convertLinesToContexts(lines)
const newDocs = newContexts.join('\n')
writeFileFromText(newDocs, './README.md')
```

`converter.js` is a module that contains
- `isCode` check this line is start with `c2e:` that mean *This line is code, right ?*
- `convertCodeToContext` convert code `c2e:` to context (code and block)
- `convertLineToContext` pass line as params and return the correct context
- `convertLinesToContexts` convert all lines to contexts (array)

```js
// converter.js
import { getTextFromFile, writeFileFromText } from './file.js'

const isCode = (line) => {
  return line.indexOf('c2e:') === 0
}

const convertCodeToContext = (line) => {
  const pathname = line.split(':').pop()
  const header = `// ${pathname}`
  const code = getTextFromFile(pathname)
  const end = "\`\`\`"
  const start = end + 'js'
  const context = [start, header, code, end].join('\n')
  return context
}

export const convertLineToContext = (line) => {
  return isCode(line) ? convertCodeToContext(line) : line
}

export const convertLinesToContexts = (lines) => {
  const contexts = lines.map((line) => {
    const context = convertLineToContext(line)
    return context
  })
  return contexts
}
```

`file.js` is a module that contains
- `getTextFromFile` receive pathname and return text
- `writeFileFromText` receive text and pathname as params and write file

```js
// file.js
import { resolve, join } from 'path'
import { readFileSync, writeFileSync } from 'fs'

const projectPath = resolve()

export const getTextFromFile = (pathname) => {
  console.log("%s %s", "Reading", pathname)
  return readFileSync(join(projectPath, pathname), 'utf-8')
}

export const writeFileFromText = (text, pathname) => {
  console.log("%s %s", "Writing", pathname)
  return writeFileSync(join(projectPath, pathname), text, 'utf-8')
}
```

```js
// converter.test.js
import { expect } from 'chai'
import { convertLineToContext, convertLinesToContexts } from './converter';
import { getTextFromFile } from './file'

describe('test converter', () => {
  it('Should convert readme to code', () => {
    const README = getTextFromFile(`./README.md`)
    const lines = README.split('\n')

    expect(convertLinesToContexts(lines)).is.an('array')
  })
  
  it('Should convert line to context if line has c2e:', () => {
    const line = 'c2e:src/main.js';
    expect(convertLineToContext(line).indexOf(getTextFromFile('src/main.js'))).to.not.equal(0)
  })
  
  it('Should return same text if line not have c2e:', () => {
    const line = '3. Convert them with command';
    expect(convertLineToContext(line)).to.equal(line)
  })
})
```

```js
// file.test.js
import { expect } from 'chai'
import { getTextFromFile, writeFileFromText } from './file'

describe('test file', () => {
  it('Should return text of file', () => {
    const path = 'README.md'
    expect(getTextFromFile(path)).to.be.a('string')
  })
  
  it('Should write file', () => {
    const text = 'test';
    const path = './src/main.js'
    const backupText = getTextFromFile(path)
    
    writeFileFromText(text, path);
    expect(getTextFromFile(path)).to.equal(text)
    writeFileFromText(backupText, path)  
  })
})
```
