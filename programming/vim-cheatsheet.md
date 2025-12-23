# vim cheat sheet

[[cheatsheet]] [[references]]

## Mode

| Command  | Description                        |
| -------- | ---------------------------------- |
| `i`      | Insert mode before the cursor      |
| `I`      | Insert mode from beginning of line |
| `a`      | Insert mode after the cursor       |
| `A`      | Insert mode from end of line       |
| `o`      | Insert mode after new line below   |
| `O`      | Insert mode after new line above   |
| `v`      | Visual mode                        |
| `Ctrl-v` | Visual-block mode                  |
| `:`      | Command line mode                  |
| `R`      | Replace mode                       |
| `Esc`    | Normal mode                        |

## Basic

| Command | Description             |
| ------- | ----------------------- |
| `w`     | To start of next word   |
| `e`     | To end of current word  |
| `$`     | To end of the line      |
| `0`     | To first of the line    |
| `^`     | To first non-blank char |

## Horizontal Navigation

| Command    | Description                        |
| ---------- | ---------------------------------- |
| `f <char>` | move on to next matching character |
| `F <char>` | move on to prev matching character |
| `t <char>` | move up to next matching character |
| `T <char>` | move up to prev matching character |
| `;`        | next character                     |
| `,`        | prev character                     |

## Vertical Navigation

| Command                | Description             |
| ---------------------- | ----------------------- |
| `{`                    | beginning of paragraph  |
| `}`                    | end of paragraph        |
| `Ctrl+d`               | half down of the page   |
| `Ctrl+u`               | half up of the page     |
| `G`                    | bottom of file          |
| `gg`                   | top of file             |
| `%`                    | go to matching bracket  |
| `:<line>`<br/>`<line>G` | go to line number       |
| `zz`                   | centre the current line |
| `(`                    | beginning of sentence   |
| `)`                    | end of sentence         |

## Other Navigation

| Command  | Description                      |
| -------- | -------------------------------- |
| `Ctrl+i` | Move to next cursor location     |
| `Ctrl+o` | Move to previous cursor location |

## Search

| Command     | Description                    |
| ----------- | ------------------------------ |
| `/ <chars>` | search chars forward           |
| `? <chars>` | search char backward           |
| `*`         | search word on cursor forward  |
| `#`         | search word on cursor backward |
| `n`         | go forward                     |
| `N`         | go backward                    |
## Search and Replace
| Command              | Description                                                      |
| -------------------- | ---------------------------------------------------------------- |
| `:s/str1/str2/g`     | Find and replace `str1` with `str2`                              |
| `:%s/str1/str2/g`    | Find and replace `str1` with `str2` in all lines                 |
| `:%s/foo/bar/gc`     | Change each `str1` to `str2` with confirmation                   |
| `:%s/\<foo\>/bar/gc` | Change exact-match `str1` to `str2` with confirmation            |
| `:%s/foo/bar/gci`    | Change each `str1` to `str2` with confirmation, case-insensitive |
| `:%s/foo/bar/gcI`    | Change each `str1` to `str2` with confirmation, case-sensitive   |
## Insert Mode

| Command | Description                           |
| --- | ------------------------------------- |
| `i` | start insert before char              |
| `I` | start insert at the beginning of line |
| `a` | start insert after char               |
| `A` | start insert at the end of line       |
| `o` | start insert bellow line              |
| `O` | start insert above line               |

## Advanced

| Command       | Mode   | Description                       |
| ------------- | ------ | --------------------------------- |
| `V`           | Normal | Select current line               |
| `vi{`         | Normal | Select all inside of curly braces |
| `va{`         | Normal | Select all around of curly braces |
| `viw`         | Normal | Select word                       |
| `viW`         | Normal | Select word including space       |
| `dit`         | Normal | Delete in html tag                |
| `vif`         | Normal | Select inside function            |
| `C-g`         | Normal | Show location and file status     |
| `<leader>d`   | Normal | Open Diagnostic                   |
| `:s/old/new`  | Normal | Find and replace                  |
| `:!<command>` |        | Shell command                     |

## Undo

| Command | Description       |
| ------- | ----------------- |
| `u`     | Undo last command |
| `U`     | Fix a whole line  |
| `C-r`   | Redo command      |

## Bookmark

| Command  | Description       |
| -------- | ----------------- |
| `m<key>` | Register bookmark |
| `<key>`  | Go to bookmark    |

## Macro

| Command  | Description              |
| -------- | ------------------------ |
| `q<key>` | Register macro           |
| `q`      | Finish registering macro |
| `@<key>` | Repeat macro             |

## Comment
| Command | Mode   | Description                      |
| ------- | ------ | -------------------------------- |
| `gc`    | Normal | Comment/Uncomment                |
| `gc`    | Visual | Comment/Uncomment selected block |

## Custom remap
| Command | Mode   | Description                     |
| ------- | ------ | ------------------------------- |
| `sv`    | Normal | Split window vertically         |
| `sh`    | Normal | Split window horizontally       |
| `se`    | Normal | Make split equal size           |
| `sx`    | Normal | Close current split             |
| `to`    | Normal | Open new tab                    |
| `tx`    | Normal | Close current tab               |
| `tn`    | Normal | Go to next tab                  |
| `tp`    | Normal | Go to previous tab              |
| `tf`    | Normal | Open current buffer in next tab |

## Surround

| Command                | Mode   | Description                                |
| ---------------------- | ------ | ------------------------------------------ |
| `ys<motion><char>`     | Normal | Add surround `<char>`                      |
| `yS<char>`             | Normal | Add surround `<char>` with `\n`            |
| `yss<char>`            | Normal | Add surround `<char>` for line             |
| `cs<charFrom><charTo>` | Normal | Change surround `<charFrom>` to `<charTo>` |
| `ds<char>`             | Normal | Delete surround `<char>`                   |
| `S<char>`              | Visual | Add surround `<char>`                      |
