# Selector Reference

What a selector matches on and how the selector engine reads it: tags per technology, matching semantics, `<nav>`, and a per-tag attribute reference (what each attribute holds, how the driver computes it).

Reference, not a procedure: targets come from [uia-configure-target](../../skills/uia-configure-target/SKILL.md) ([ui-automation-guide.md § Mandatory](../../ui-automation-guide.md#mandatory-generate-targets-before-writing-any-ui-code)), and attribute reliability is ranked in [selector-improvement-guide.md](selector-improvement-guide.md). Read it to interpret a selector, to explain why one matches or misses, or to choose among attributes a live listing printed. Against a live application the `selector-intelligence` commands outrank every table here (§ [Live commands](#live-commands)).

## Tags per technology

A selector is a chain of tags: one top-level tag for the window or browser tab, then one tag per ancestry level needed to identify the element. The element's technology decides the tags. Tags of two technologies sit under one window only where the technology changes (browser chrome and page content, a Java applet inside a browser).

| Technology | Top-level tag | Element tag | Notes |
|---|---|---|---|
| Browser page (Chrome, Edge, Firefox) | `<html app='chrome.exe' title='…' url='…' />` | `<webctrl … />` | `app` is the browser executable |
| SAP web (Fiori/UI5, Web GUI, Web Dynpro, Ariba) | `<html … />` | `<webctrl … />` with `ui5-*`, `sapweb-*`, `aw-*` attributes | |
| SAP GUI for Windows | `<wnd app='saplogon.exe' cls='SAP_FRONTEND_SESSION' title='…' />` | `<sap id='…' />` | modal dialogs are windows of their own (`cls='#32770'`) |
| SAP GUI for Java | `<wnd app='java*.exe' cls='SunAwtFrame' title='…' />` | `<sap id='…' />` | dialogs `cls='SunAwtDialog'`; custom Java classes surface as `<java>` |
| Windows desktop through UI Automation (WPF, WinForms, Win32, UWP, Electron shells, browser chrome) | `<wnd app='<exe>' cls='…' title='…' />` | `<uia … />` | |
| Windows desktop through Active Accessibility | `<wnd … />` | `<ctrl … />` | |
| Win32 child windows (controls that own a window handle) | `<wnd … />` | nested `<wnd cls='…' ctrlid='…' ctrlname='…' />` | |
| Java Swing/AWT | `<wnd app='java*.exe' cls='SunAwtFrame' title='…' />` | `<java … />` | |
| Qt | `<wnd … />` | `<qt … />` | |
| Remote desktop through the UiPath remote runtime (Citrix, RDP, VMware) | `<rdpapp … />` / `<rdp … />` | native tags of the remote application | `<rdp>` carries the `<wnd>` attribute set |
| macOS | `<wnd app='…' title='…' />` | `<ax … />` | |
| Relative navigation | | `<nav up='n' />`, `<nav next='n' />`, `<nav prev='n' />`, `<nav child='k' />` | § [`<nav>`](#nav) |

A top-level `<wnd>` or `<html>` needs `app`: without it an `<html>` is Internet Explorer, and a `<wnd>` matches any window its other attributes match. An `<html>` tag is resolved among the browser's tabs whatever tag precedes it: a `<wnd>` before it narrows nothing, though it must still match a window, and a page's window selector is the `<html>` tag alone. A browser's own UI (tab strip, toolbar, native dialogs) is `<uia>` under the browser's `<wnd>`, a window selector separate from the page's.

## Matching semantics

Properties of the selector engine.

**Values match whole.** A value with no wildcard must equal the element's value exactly; no `*` is implied at either end. `*` matches zero or more characters, the empty string included; `?` matches exactly one. There is no escape character: a literal `*` or `?` needs `matching:<attr>='regex'`. On desktop technologies carriage returns are normalised on both sides before comparison (`\r\n` and `\r` become `\n`); the browser compares `webctrl` and `html` values as they are.

**Case.** Attribute values are case-sensitive, except `app` on every tag, `tag` on `webctrl`, `role` on `java` and `enabled`, `focused`, `selected`, `expanded` on macOS; `casesensitive:<attr>='false'` lifts it per attribute. Attribute names are case-sensitive on every tag except `webctrl` and spelled as the attribute references spell them (`tableRow`, `colName`, `colTooltip`, `rowName`, `labeledby`, `htmlwindowname`): a mis-cased name is a parse error, not a silent miss. `webctrl` reads an attribute name in any case. Tag names are case-insensitive.

**Schema.** Closed for every tag except `webctrl`: an attribute its tag's reference does not list is a parse error. `webctrl` also takes any HTML attribute of the element, and a name the element does not carry reads as empty.

**Matching modes**, per attribute with `matching:<attr>`:

| Mode | Behaviour |
|---|---|
| (absent) | wildcard matching as above |
| `regex` | ECMAScript, matched against the whole value (anchored). An invalid pattern is a parse error, and so is an inline flag (`(?i)`); case is set with `casesensitive:<attr>`. On `webctrl` and `html` the browser anchors the pattern as `^<pattern>$` without a group, so a top-level alternation needs one: `(Save\|Submit)`, not `Save\|Submit` |
| `fuzzy` | similarity score compared with the threshold `fuzzylevel:<attr>` |

- `casesensitive:<attr>='false'` applies in `regex` mode too.
- In a pattern `.` does not cross a line break and `[\s\S]` does; a wildcard `*` crosses line breaks.
- `fuzzylevel:<attr>` is the threshold the score must reach, `0.0` to `1.0`, default `1.0`. An imperfect match never scores above 0.999, so `matching:<attr>='fuzzy'` without a `fuzzylevel` is stricter than plain equality, and `fuzzylevel:<attr>='0.0'` always matches on that attribute, which is what a fuzzified selector carries.

**`idx`** is 1-based, counted in document order over the elements that match every other attribute of its tag, within the scope of the preceding tag: descendants, not siblings. Without it the first match is taken. Fuzzifying a selector strips `idx`, `tableRow`, `tableCol` and `css-selector`, and a fuzzy selector does not support `idx` ([Target](../../activities/common/Target.md)).

**Greedy search.** A tag before the last follows only its first match, unless it carries a fuzzy attribute: when the rest of the selector fails under that match, the search fails without trying the tag's other matches. `idx='*'` marks a tag non-greedy, so the rest is tried under each of its matches; `evaluate` writes it on a top-level tag that matches several windows.

**Number attributes** take a plain integer, digits with an optional leading `-`: a wildcard, range or comparison fails to parse. Among them: `idx` on every tag (`idx='*'` aside), `pid` on `wnd`, `html` and `rdp`, the `<nav>` counts, `tableRow` and `tableCol` on `uia`, `java`, `sap`, `curl` and `ax`, and `rowCount` and `columnCount` on `ax`. On `uia` `pid` is text, and on `webctrl` every attribute but `idx` is text, `tableRow` and `tableCol` included. An empty value is zero, never "any": an `idx` of zero matches nothing on `wnd`, `uia`, `ctrl` and `java`, and is the first tab on `html`.

**The selector is XML.** Values are quoted, single quotes by convention; `&apos;`, `&amp;` and `&lt;` escape the characters that need it; a repeated attribute on one tag rejects the whole selector. `attr=''` is an active criterion that matches only an empty value; the prefix `omit:` carries an attribute without matching on it. The driver accepts no modifier besides `omit:`, `matching:`, `fuzzylevel:` and `casesensitive:`, and sets no cap on attributes per tag.

**`check:<attr>`** belongs to the activities layer: only the last tag of a fuzzy selector or of an anchor carries it, and a strict selector that names it does not parse. `<attr>` says how the live text is read: `text`, `innerText`, `visibleinnertext`, `aaname`, `fulltext` or `native`. After the fuzzy match it drops every candidate whose live text does not match the value as a case-insensitive wildcard pattern; the live text is trimmed and its line breaks normalised, and on targets of version V3 and earlier it is cut at a word boundary after 100 characters. Approximate matching skips the check. Generation writes it on every anchor, and on the targets of every activity except Get Text, Type Into, Set Text and Select Item, on any element but an input box, a password box, a drop-down or a cell; in browsers it is `check:innerText`.

**Long values.** Generation and snapshot capture truncate a long value and end it with `*`, which then matches as a wildcard against the full live value:

| Limit | Attributes |
|---|---|
| 512 | `webctrl`, except `id`, `href`, `src`, `css-selector` and `sfl-*` |
| 256 | `html` `htmlwindowname` |
| 64 | `wnd` `title`, `aaname`, `cls`; `ctrl` `automationid`, `name`, `labeledby`; `uia` `helpText`, `legacyAccHelp`; `wpf`, except `type`, `name`, `uid`, `automationid` |

## `<nav>`

Moves through the element tree (the hierarchy a snapshot shows, never screen position) from the node the preceding tag matched. One of `up`, `next`, `prev`, `child` per tag:

| Step | Lands on |
|---|---|
| `up='n'` | n-th ancestor; `n` at least 1 |
| `next='n'` / `prev='n'` | n-th following / preceding sibling, hidden ones included; `n` at least 1 |
| `child='k'` | k-th child; `child='-k'` is the k-th counted from the last (`-1` the last); any count but 0 |

- Never first in a full selector. A partial selector may start with one, which then moves from the window its scope matched.
- The landed node's tag is not checked. A `<nav>` that ends the selector makes the landed node the target; a tag after it is searched among the landed node's descendants, never the landed node itself.
- `<nav>` tags chain: `<nav up='1' /><nav next='1' />`.
- The tag before a `<nav>` is followed from its first match only, as every tag before the last (§ [Matching semantics](#matching-semantics), greedy search).
- A step past the first or last sibling or child fails the search, and so does a count below 1 on `up`, `next` or `prev`.
- A control that is a window of its own (a Win32 or Windows Forms control) has that window as its parent, a level the snapshot does not show: `up='1'` lands on the window, and `next` / `prev` from the control count the window's own children (title bar, scroll bars). The control beside it is `<nav up='1' /><nav next='1' />` from it, or a `child` of their container.
- An empty tag after a node lands on that node's first child of the tag's technology. `idx` counts matching descendants, not children, so a child position is a `child` step, not an `idx`.

## Live commands

What the `selector-intelligence` commands report and reject (syntax: [cli-reference.md](../cli-reference.md)):

- `get-selector-attributes` prints, from the snapshot, the attributes the driver captured for one node with a non-empty value. Author selectors from this list. An attribute missing from it may still exist: one that was empty at capture (`attr=''` matches it), and on `webctrl` an attribute the browser computes on demand, such as `labeledby`, `innertext`, `checked` or `readonly` ([webctrl-attributes.md](tag-attributes/webctrl-attributes.md)). A name a tree view shows for the node is no attribute.
- `evaluate` runs against the snapshot, never the live application. It removes every `idx`, lists up to 10 nodes the selector then matches, hidden ones included when the snapshot holds them, and rejects an attribute its node does not carry. A `<nav>` tag fails it too, since `up`, `next`, `prev` and `child` are no node attributes: check a selector with a `<nav>` step by validating the target against the live application.

## Attribute references

| Tag | Attributes | Reliability ranking |
|---|---|---|
| `wnd` | [wnd-attributes.md](tag-attributes/wnd-attributes.md) | [wnd-instructions.md](tag-names-instructions/wnd-instructions.md) |
| `html` | [html-attributes.md](tag-attributes/html-attributes.md) | [html-instructions.md](tag-names-instructions/html-instructions.md) |
| `webctrl` | [webctrl-attributes.md](tag-attributes/webctrl-attributes.md) | [webctrl-instructions.md](tag-names-instructions/webctrl-instructions.md) |
| `uia` | [uia-attributes.md](tag-attributes/uia-attributes.md) | [uia-instructions.md](tag-names-instructions/uia-instructions.md) |
| `java` | [java-attributes.md](tag-attributes/java-attributes.md) | [java-instructions.md](tag-names-instructions/java-instructions.md) |
| `sap`, SAP session attributes on `wnd` | [sap-attributes.md](tag-attributes/sap-attributes.md) | |
| `ctrl` | [other-tags-attributes.md](tag-attributes/other-tags-attributes.md) | [ctrl-instructions.md](tag-names-instructions/ctrl-instructions.md) |
| `wpf`, `silverlight`, `qt`, `curl`, `ax`, `rdp`, `rdpapp` | [other-tags-attributes.md](tag-attributes/other-tags-attributes.md) | |
