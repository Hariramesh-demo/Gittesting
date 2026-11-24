# `uia` attributes

UI Automation element tag. Part of [selector-reference.md](../selector-reference.md); reliability ranking: [uia-instructions.md](../tag-names-instructions/uia-instructions.md).

| Attribute | Holds | Notes |
|---|---|---|
| `automationid` | AutomationId | |
| `role` | control type, lower-case (`button`, `combo box`, `edit`) | language-agnostic: same value in every UI language |
| `cls` | class name | |
| `name` | Name | |
| `helpText`, `legacyAccHelp`, `legacyAccDescription` | help text | |
| `tableCol`, `tableRow`, `rowName`, `colName` | grid position and headers | |
| `roleint` | control type id | |
| `enabled`, `ispwd`, `kbfocus`, `itemstatus`, `itemtype`, `accelkey`, `acskey` | state, keys | |
| `rtid`, `pid` | runtime id, process | change with every run |
| `idx` | position among matches | |

## Control type → `role`

| Control type | `role` | Control type | `role` |
|---|---|---|---|
| Button | `button` | Edit | `edit` |
| CheckBox | `check box` | Text | `text` |
| RadioButton | `radio button` | Hyperlink | `hyperlink` |
| ComboBox | `combo box` | Image | `image` |
| List / ListItem | `list` / `list item` | Tree / TreeItem | `tree` / `tree item` |
| Tab / TabItem | `tab` / `tab item` | Menu / MenuBar / MenuItem | `menu` / `menu bar` / `menu item` |
| DataGrid / DataItem | `data grid` / `data item` | Table / Header / HeaderItem | `table` / `header` / `header item` |
| ToolBar / StatusBar / ProgressBar / ScrollBar | `tool bar` / `status bar` / `progress bar` / `scroll bar` | TitleBar / Window / Pane / Group / Document / Custom | `title bar` / `window` / `pane` / `group` / `document` / `custom` |
| Slider / Spinner / SplitButton / Thumb / ToolTip / Separator | `slider` / `spinner` / `split button` / `thumb` / `tool tip` / `separator` | | |

## Chromium content through UI Automation

Chromium-rendered content reached through UI Automation (an Electron renderer window, browser chrome, a web view without the browser driver) exposes the page's accessibility tree, not its DOM. `role` is the control type Chromium derives from the element's implicit or explicit ARIA role; `name` is its accessible name (associated label, `aria-label`, own text, placeholder as the last fallback). DOM `class`, `data-*`, `name` and `type` attributes are not accessibility properties and never reach the element. Whether `automationid` mirrors the DOM `id` varies: read it from the live attribute listing.

## Windows Forms list view

A Windows Forms list view (`SysListView32`) exposes each row as a `list item` whose cells are `text` children, and its column headers as `header item`s under a `header`, outside the rows. Each cell carries `colName`, its column's header caption, and `rowName`, the text of its row's first cell: a cell by row content and column caption is `<uia role='text' rowName='<key>' colName='<header caption>' />` under the list. Searched from the window instead of the list, `<uia role='text' />` also takes the window's captions and those of the windows it owns.

## Owned windows

A window's UI Automation tree holds the windows it owns. A modal dialog, a form the window opened and the windows those own in turn sit under their owner, topmost first and ahead of the owner's own elements. A `uia` search from the owner's scope therefore also matches in every open window it owns, and there first, whatever the Application Card's attach mode: a Find Elements with no target returns the owned windows' matches ahead of the owner's, and a target under the owner's window selector can resolve a control of an owned window.
