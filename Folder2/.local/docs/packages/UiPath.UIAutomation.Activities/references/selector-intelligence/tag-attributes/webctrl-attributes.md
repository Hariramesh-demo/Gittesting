# `webctrl` attributes

Element tag of pages driven through the browser driver: browsers, web views, SAP Fiori, Web GUI and Ariba. Open schema: besides the attributes below, any HTML attribute of the element matches under its own name. Part of [selector-reference.md](../selector-reference.md); reliability ranking and value hygiene (ids, classes, framework attributes): [webctrl-instructions.md](../tag-names-instructions/webctrl-instructions.md).

## Search

A `webctrl` search visits every element of the page in document order, a shadow root's content before its host's other children, hidden elements included, and takes the first that matches.

- **Shadow DOM.** A tag whose node lives in a shadow root must carry `shadowhostid`: a tag without it never matches a node inside one. In a generated selector the host also gets a tag of its own.
- **Frames.** One `<webctrl tag='IFRAME' … />` (`FRAME`, `EMBED`) per frame level, carrying `id`, `name`, `src` (never `class`); nested frames give one tag per level. There is no `frame` attribute. SAP web frames get no frame tag.
- **`idx`.** 1-based, document order, counted over the elements in the subtree of the preceding tag's element that match the tag's other attributes, not among siblings.

## Attributes

How the web extension computes each attribute:

| Attribute | Computed as | Consequence |
|---|---|---|
| `tag` | DOM tag name, upper-case for HTML elements | matched case-insensitively |
| `aaname` | legacy MSAA-style name, **not** the ARIA accessible name. For `A`, `INPUT`, `IMG`, `BUTTON`, `TEXTAREA`, `AREA`, `SELECT`, `TABLE`: buttons `title` → text/value; images `title` → `alt` → `<label for>` (→ enclosing `A`); tables `title` → caption → header cells; other form elements and links `<label for>` text → own text/value → `title` → `alt`. For every other element: its own direct text nodes only. Text is taken as the node holds it, not trimmed, inner whitespace kept: a link indented in the markup reads `'      Travel      '` while its `visibleinnertext` reads `'Travel'` | never reads `aria-label`, `aria-labelledby`, `placeholder` or a wrapping `<label>`; not inherited from descendants (§ [Which nodes carry `aaname`](#which-nodes-carry-aaname)); an input's `aaname` is its `<label for>` text when the input has an `id` |
| `value` | the HTML `value` attribute, read raw, not the live `.value` property: typing into a field or assigning `.value` from script leaves it as the page served it (empty when the markup has none), only `setAttribute` changes it. On an `INPUT` of type submit, button or reset, assigning `.value` writes the attribute, so it always holds the caption. On a `SELECT` it is empty unless the markup sets it | never the content typed into a field (that is `text`) and never a `SELECT`'s selection; on a button with a `title`, `aaname` is the title and `value` the caption |
| `innertext` | `textContent`: all descendants, hidden ones included; newlines become spaces; no trim, no whitespace collapsing | every ancestor's value contains it |
| `visibleinnertext` | the browser's `innerText`, returned as is: the rendered text, hidden descendants excluded, a line break between blocks and between table rows, a tab between table cells. The driver neither trims nor collapses it | every ancestor's value contains it. A table row carries the tabs of its empty cells (`"\t\tSystem\t\t   "`). A line break in a value is written `&#10;`; a literal one becomes a space when the definition file is read |
| `outertext` | identical to `innertext` | |
| `title` | the **document** title | the element's `title` attribute is `uipath-html-title` |
| `aria-role`, `aria-label`, `aria-labelledby`, `type`, `placeholder`, `href`, `src`, `alt`, `data-*` | raw DOM attribute, as the page wrote it, padding included. `aria-role` is only a written `role`, never an element's implicit one; `aria-labelledby` is the ids it points to, not their text | `role` on `webctrl` is the constant `element`; the written role is `aria-role` |
| `labeledby` | the element's `aria-label` as the page wrote it, else the trimmed text of a `LABEL` in the page's document, hidden text included: the one its `aria-labelledby` names when that is a single id, or the one whose `for` names its `id`. Never a `<label>` around the element, a label inside a shadow root, an `aria-labelledby` of several ids or one naming another element, or the `title` | a field's name; empty for a field named only in those ways, so a field inside a shadow root has one only from its `aria-label`. `get-selector-attributes` leaves it out; selectors match it |
| `parentid`, `parentname`, `parentclass` | nearest ancestor with a non-empty `id` / `name` / `class`; crosses shadow roots | not the direct parent |
| `isleaf` | `1` when the element has no element children; an element holding only text counts as a leaf | |
| `uipath-visible` | `1` when the element is rendered: not `display: none`, visibility `visible`, not clipped to nothing, a box with width and height above 0 unless it is `display: contents`; else `0` | the one attribute that leaves hidden elements out of a match |
| `tableRow`, `tableCol`, `colName`, `rowName` | real `TABLE` / `TR` / `TD` / `TH` markup only; 1-based, header rows counted (under one header row the first data row is `tableRow` 2). `colName` is the full text of the header cell at the same column position, visually hidden text included, such as a sort caption (`Document CategorySort and filter column`). `rowName` is the text of the row's header cell; a multi-line header concatenates without separators (`Capitaland largest city`). Both hold the text as the markup holds it, untrimmed: indented markup pads them (`'                     Sub-Total:                 '` for a cell that renders `Sub-Total:`), compact markup does not | absent on div-based grids (`role='grid'`); a bare visible caption does not match a header that carries hidden text or padding |
| `css-selector` | generated as a tag-name-only chain from `body` (`body>div>span>a`), no ids or classes. Matched as a CSS selector on the candidate element (the browser's `Element.matches`) | any selector the browser accepts is a criterion, positions included (`:nth-of-type`, `:nth-child`) |
| `shadowhostid` | `id` of the shadow host of the element's root; empty when the host has no `id`, and for an element outside any shadow root | `'*'` matches under any host and outside shadow roots alike; `''` matches outside shadow roots and under a host that has no `id` |

## Which nodes carry `aaname`

| Node | `aaname` |
|---|---|
| Form-ish tag: `A`, `INPUT`, `BUTTON`, `TEXTAREA`, `SELECT`, `IMG`, `AREA`, `TABLE` | synthesised from `<label for>`, `title`, `alt`, text or value (table above). A labelled `INPUT`'s `aaname` is its label text |
| Leaf that owns its text: a `DIV` / `SPAN` / `LI` / `LABEL` / `TH` with `isleaf='1'` and a text node of its own | that text. `aaname` with `isleaf='1'` also matches the form control a `LABEL` names (a hidden radio beside its caption), which carries the label's text as its own `aaname`; `tag` tells the two apart |
| Container wrapping other elements (`isleaf='0'`) | only its own direct text, so usually empty or whitespace. The text a container shows is its descendants' and sits in `visibleinnertext` / `innertext`; its own name, when it has one, is `aria-label` or `aria-labelledby` |

## Generated selectors

Attribute preference of the driver when it generates a `webctrl` tag, in order; stage one is always emitted, later stages are added only while they lower the index: `shadowhostid` (shadow DOM), `tag`, `tableRow`, `id`; `type`; `aria-role`; `aria-label`; `aria-labelledby`; Salesforce `sfl-*`; Ariba `aw-*`; Oracle ADF `adf-*`; `name`; `src` (frames); `parentid`; `parentname`; `class`; `aaname`; `isleaf`; `tabindex`; `tableCol`; `href`; `src`; `css-selector`.

Volatility filters:

- An `id`, `parentid` or `aria-labelledby` containing a letter after a digit (`_oj146_smart-filter`, `input-45f6g7h8`) is rejected as auto-generated.
- A `class` with several tokens is never used.
- `aaname` is offered only for `a`, `label`, `img`, `input`, `button`, `textarea`, `area`, `select`, `table`, `th`, for a `div` / `span` whose text is 32 characters or shorter, or under such an ancestor. The offer also depends on the activity type: a Click default may prefer a positional index over a multi-word caption that a `None` default keeps.

SAP Fiori and Web GUI elements bypass this order and use their own attribute families.

## SAP web identifiers

| Identifier | What it is |
|---|---|
| SAP Web GUI DOM `id`, repeated as `sapweb-lsid` on every element that is one of the page's controls: a field's `M0:46:::1:34`, a toolbar button's `M0:50::btn[8]` | on a transaction screen's fields, labels and toolbar buttons, the control's place on its screen: window (`M0` the main window, `M1` the first pop-up), area, then a field's row and column or a button's function code. The same screen gives the same id on every system; another layout of the screen gives other ids. The window and trees have ids of other forms (`webguiPage0`, `tree#C105#1#1#1#i`) |
| `sapweb-id`: `wnd[0]/usr/ctxtMS_MATNR-LOW` | the SAP GUI scripting id: window, area, any subscreen and tabstrip containers, control-type prefix and the screen field's name. SAP GUI's `<sap id>` is the same path after the window (`usr/ctxtMS_MATNR-LOW`). A field's `sapweb-id` is not derivable from its DOM `id` |
| SAP Fiori DOM `id`: `<view id>--<local id>`, the view id carrying the application (`application-…-component---…view.Main`, `…::…ListReport::C_PurchaseOrderTP`) | `ui5-view-local-id` is the part after the view's `--`. An id ending `-inner` is the `INPUT` inside the control whose id precedes it: the control's `ui5-*` attributes, `ui5-label` among them, sit on the control's element, and the field's `labeledby` on the inner `INPUT` |
