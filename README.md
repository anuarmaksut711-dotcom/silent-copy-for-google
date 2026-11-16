# Google Code Cleaner

Version 1.1.0 · Copyright (c) 2026 Anuar Maksut

Google Code Cleaner is a browser extension that copies code from Google AI answers and Gemini to your clipboard and removes supported code comments. Automatic copying is silent: it adds no buttons, notifications, text selection, or other visible changes to the Google page.

The extension copies text. It does not execute the copied code or insert it into a website.

## Install

1. Extract the downloaded ZIP into a permanent folder.
2. Open `chrome://extensions` in Chrome, or `edge://extensions` in Edge.
3. Turn on **Developer mode**.
4. Click **Load unpacked** and select the folder containing `manifest.json`.
5. Reload any Google or Gemini tabs that were already open.

The manifest requires Chrome 116 or later. Other Chromium browsers must support the extension APIs it uses. Keep the extracted folder after installation: an unpacked extension loads its files from that folder.

This package is ready to load. It does not require Node.js, a terminal, a server, an API key, a build step, or installation of additional dependencies.

## Use

1. Ask Google or Gemini for code.
2. Wait for the answer to contain a visible code block. Expand a collapsed answer yourself if necessary.
3. Keep that browser tab active and focused while the code finishes appearing.
4. After the configured pause, the extension copies the code automatically.
5. Switch to your editor and press **Ctrl+V** to paste it.

**Automatic copying is enabled by default and replaces the current clipboard contents.** There is no automatic-copy notification. The copied text remains available until you or another application replaces or clears the clipboard.

## Settings

Click the extension icon to open its settings. The interface currently uses Russian labels; their meanings are listed below.

| Control | Meaning | Default |
| --- | --- | --- |
| `Автокопирование` | Enable or disable automatic copying | Enabled |
| `Блоки кода` | Copy the last block, or all blocks in page order | Last block |
| `Пауза` | Wait after a captured code update before copying | 1.8 seconds |
| `Скопировать сейчас` | Copy the current code manually | Button |

The pause can be set to **1.8, 3, or 5 seconds**. Settings are saved locally. Manual copying remains available when automatic copying is disabled. If the answer is still generating, manual copying asks you to wait.

In all-blocks mode, each block is cleaned separately and the results are joined with a blank line. HTML, CSS, and JavaScript are not assembled into one executable file; paste each fragment into its appropriate destination.

## How copying works

1. **Find the answer.** On a supported Google search page, the extension looks for an AI Overview container or a recognized AI Overview heading. In Google's recognized AI Mode URLs, it uses the main page area. In Gemini, it selects the last visible model response in page order.
2. **Find code blocks.** It looks for `pre`, `code-block`, `[role="code"]`, and `[data-code-block]` elements inside that area. Hidden, empty, and duplicate nested blocks are skipped. A short inline `code` element inside a sentence is not selected on its own.
3. **Wait for stable code.** Relevant changes are grouped with a short 250 ms delay. A new snapshot then starts the configured pause. The extension checks available generation indicators and reads the code again before sending it to the background process.
4. **Remove comments in the background.** Language-specific rules clean the selected blocks. JavaScript parsing runs in the extension service worker rather than Google's page thread.
5. **Write to the clipboard.** A hidden extension document writes the prepared text using a temporary text field. It clears that field afterwards without selecting text on the Google page.

**The 1.8-second setting is a waiting period, not a repeating copy interval.** A typical new code update takes approximately 250 ms plus the selected pause, followed by processing time. Browser scheduling can add delay.

If a recognized generation indicator is present, copying is postponed; the indicator may be checked again every 1.5 seconds. After successful copying, the pending timer stops. The extension does not repeatedly scan the whole page every 250 ms or repeatedly copy unchanged code.

The last automatically copied snapshot is remembered in tab memory and compared with later snapshots. The comparison includes the page address, answer identity, selection mode, code text, and language labels. Reloading the page resets this memory. A new answer or changed code can be copied again.

The extension does not read your clipboard. If you copy other text afterwards, an unchanged old answer is not restored on a timer. Manual copying has its own path and does not update the automatic-copy marker, so an answer not yet copied automatically may still be copied after you close the settings window.

## Comment removal

| Language | Supported comment handling |
| --- | --- |
| JavaScript / JSX | Parser-recognized line, block, and JSX comments; strings, URL strings, regular expressions, and template text are preserved |
| HTML | HTML comments and supported comments inside embedded JavaScript and CSS; attribute values and textarea/title text are preserved by the handler |
| CSS | Block comments outside strings and URL contents |
| Python | Basic hash comments outside ordinary or triple-quoted strings; docstrings are retained |
| JSON / JSONC | Line and block comments; string values are preserved |
| SQL | Line and nested block comments, with supported quoted strings/identifiers and dollar-quoted strings |

JavaScript and JSX use the bundled Acorn and acorn-jsx parsers. Other languages use dedicated scanning rules, not full compilers or validators for every dialect.

TypeScript, TSX, Bash, Java, PHP, Rust, and other unsupported labels are copied **without comment removal**. If parsing or scanning fails, the entire block is kept without partial cleaning. Python blocks detected as containing f-strings, and SQL blocks with backslashes in processed quoted strings or identifiers, are also left uncleaned. Without a language label, language detection is approximate.

Line endings are normalized to LF, including blocks left uncleaned. Comments are usually replaced with spaces while line breaks are preserved. HTML comments are removed without inserting spaces between adjacent elements. Blank lines and extra spaces may remain: this is not a formatter or minifier for the copied code.

License notices and build-tool directives inside copied comments are not specially protected. Review the result when those comments matter to your project. The extension's own licenses are included separately in this distribution.

Manual copying shows a message if some code could not be cleaned. Automatic copying stays silent, so unsupported code may reach the clipboard with comments still present.

## Supported pages and limits

- HTTPS searches on `google.com`, `google.kz`, and `google.ru`, with or without `www`, at paths `/`, `/search`, and `/webhp`.
- HTTPS pages on `gemini.google.com`.
- Automatic copying runs only while the supported page is active and focused. Page observers and pending timers stop when it loses focus, goes into the background, or automatic copying is switched off. They resume when appropriate.
- Processing is limited to 128 selected blocks and 1,000,000 UTF-16 code units of input per operation. The final clipboard text must also fit within 1,000,000 code units and contain non-whitespace text. This is a string-length limit, not a one-megabyte limit.
- Google must actually provide a visible code block. The extension does not generate missing code, expand answers, or extract code from images or PDFs.

Answer detection depends on Google/Gemini markup and recognized headings. Site changes can require an extension update. Recognizing the end of generation is approximate: a long pause without a recognized busy indicator can cause an intermediate version to be copied. Later code changes start another attempt. A short asynchronous gap also exists between the final check and the clipboard write.

Page reading, visibility checks, and messaging still use resources. Background parsing and filtered observation reduce unnecessary work; they do not guarantee zero overhead on every device.

## Privacy and permissions

Processing is local. The extension does not send code to a server or external AI API, does not include analytics, and does not store a permanent answer history. Settings are saved in `chrome.storage.local`; current and recently copied snapshots may be held temporarily in memory.

The requested permissions are:

- **storage:** save settings.
- **clipboardWrite:** write prepared code to the clipboard.
- **offscreen:** use a hidden extension document for clipboard access without moving focus to a visible window.

The extension does not request `clipboardRead`. Its page access is limited to the Google/Gemini addresses declared in the manifest. Operating-system clipboard history, if enabled, is managed separately by the operating system.

## Update or troubleshoot

To update, replace the files in the installed folder, open the browser's extensions page, click the reload icon on the Google Code Cleaner card, and reload your Google/Gemini tabs. An already-open page keeps its old content script until that page is reloaded.

If nothing is copied, check that the extension is enabled, automatic copying is on, the domain is supported, the page has focus, and a visible code block exists. Try the manual copy button to see an error or empty-result message. For a slowly generated answer, try a longer pause.

If you install from a different folder, remove the old installation before loading the new folder to avoid running two copies. You can disable or remove the extension from the browser's extensions page.

## License

The extension's own code is licensed under the **MIT License**. See [LICENSE](LICENSE).

Copyright (c) 2026 Anuar Maksut.

Bundled third-party copyright and license notices for Acorn and acorn-jsx are retained in [lib/THIRD-PARTY-LICENSES.txt](lib/THIRD-PARTY-LICENSES.txt).
