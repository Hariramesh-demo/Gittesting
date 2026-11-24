# SAP GUI attributes

SAP session attributes on `wnd` and the `sap` tag, for SAP GUI for Windows and SAP GUI for Java. Part of [selector-reference.md](../selector-reference.md); authoring guidance: [sap-guide.md](../../sap-guide.md). SAP web (Fiori, Web GUI, Ariba) is `webctrl`: [webctrl-attributes.md](webctrl-attributes.md).

`<sap>` ids are the same on both front ends. `type` values differ in case, and SAP GUI for Java keeps the inactive system-toolbar buttons that SAP GUI for Windows leaves out of the tree.

## Session window

On `wnd`, beside its own attributes ([wnd-attributes.md](wnd-attributes.md)):

| Attribute | Holds | Notes |
|---|---|---|
| `sapSysSessionId` | server-side SAP session id, on the session window and on every SAP modal dialog of that session; absent on the SAP Logon pad | runtime value, matched as "present": `sapSysSessionId='?*'` separates SAP dialogs from the other `#32770` windows of `saplogon.exe` |
| `sapSysName`, `sapClient`, `sapUser`, `sapLanguage`, `sapTransaction`, `sapProgram`, `sapScreen`, `sapWindowName` (`wnd[0]`, `wnd[1]`), `sapGuiSessionId` (`/app/con[0]/ses[0]`), `sapSession`, `sapSysNumber` | SAP GUI session context | `sapScreen`, `sapSession`, `sapSysNumber` are numbers: `sapScreen='0100'` matches screen 100. `sapTransaction`, `sapProgram` and `sapScreen` change with every screen, as `title` does |
| `sapReadyState` | whether the session is idle | Get Attribute only: a selector that names it does not parse |

- The session window's `title` is the current screen's (`SAP Easy Access`, a transaction screen's caption). The system name, session number and client a title can also show (`<SID> (<session>) (<client>)`) change with the logon.
- A modal dialog is a window of its own: `cls='#32770'` (SAP GUI for Java: `SunAwtDialog`), `sapSysSessionId` present, `title` its caption. Its `sapProgram` and `sapScreen` are the popup's own program and screen number.
- A dialog's controls carry the same `<sap id>` as the session window's (`tbar[0]/btn[0]`, `usr/…`), and `usr/…` resolves inside whichever window the chain names: the window tag tells them apart.

## `sap`

| Attribute | Holds | Matching |
|---|---|---|
| `id` | normalized scripting id: everything after `wnd[n]/` (`usr/ctxtRF02D-KUNNR`, `tbar[1]/btn[8]`, `usr/cntlGRID1/shellcont/shell`). `/app/con[0]/ses[0]/` and the `wnd[n]/` segment are stripped; `usr/` stays. The driver's default target for any activity type is this `id` alone, strict | plain, case-sensitive text resolves in one scripting call (`FindById`). An id with `wnd[n]/` left on, with `usr/` cut, or reduced to the control's last segment matches nothing. A wildcard, a `regex` / `fuzzy` matching mode or `casesensitive:id='false'` moves the search onto a per-node tree walk with no node cap and no timeout of its own. `*` and `?` match within one `/`-separated segment and never cross a `/`: `id='*/okcd'`, `id='tbar[0]/ok*'` and `id='usr/*/*/*/*/shell'` resolve, `id='*okcd'` and `id='usr/*/shellcont[0]/shell'` do not, and a segment wildcard resolves to whichever node first fills the pattern (`id='usr/cntlIMAGE_CONTAINER/*'` is the container's first child). A field by its technical name at any depth is the regex form `id='.*<prefix><FIELD>' matching:id='regex' type='Gui…'`, with `[` and `]` in the tail escaped as `\[` `\]`. A backslash in an id (`tabpT\01`) stays literal under segment wildcards (`usr/*/tabpT\01`) and is doubled in the regex form (`id='.*tabpT\\01' matching:id='regex'`); a single one there fails to parse |
| `leafid` | the `id`'s last segment: type prefix, name, array suffix (`ctxtRF02D-KUNNR`, `okcd`, `btn[8]`) | matched at any depth below the preceding tag, over the same tree walk as a wildcard `id`: a leaf several nodes share (`shell`, `shellcont`) resolves to the shallowest. Beside `id` it fails the search. The driver never generates it |
| `type` | SAP component type name (`GuiCTextField`, `GuiButton`, `GuiTableControl`). The driver names the ALV grid and tree `GuiGridView` and `GuiTree`, and their parts `GuiGridViewCell`, `GuiGridViewRow`, `GuiGridViewColumn`, `GuiTreeNode`, `GuiTreeItem` | case-sensitive, and the case is the front end's: SAP GUI for Windows reports the scripting API's names (`GuiOkCodeField`, `GuiStatusbar`, `GuiTitlebar`), SAP GUI for Java its own (`GuiOKCodeField`, `GuiStatusBar`, `GuiTitleBar`), so a type from one front end matches nothing on the other. A plain-text `id` resolves without it; beside a wildcard or regex `id` it narrows the tree walk, on the elements that take it (table below) |
| `text` | the `Text` property, untrimmed (SAP pads some captions with spaces); a grid cell reports its cell value | beside a column attribute (`tableCol`, `colName`, `colTooltip`) and without `tableRow`, finds the first cell of that column holding the value: a row by content |
| `tooltip` | `Tooltip`, falling back to the accessibility tooltip, then the default quick-info; untrimmed. On input fields the quick-info usually carries the field label | |
| `tableRow` | 0-based **absolute** row of a table control or ALV grid, not visible-relative; addressing a row may scroll | alone, the row element, whose `checked` attribute is the row's selection |
| `tableCol` | 0-based **absolute** column of a classic `GuiTableControl`, not the displayed one | alone, the column element; with `tableRow`, the cell. `tableCol='-1'` with a `tableRow` is the row-selector cell. `tableRow='-1' tableCol='-1'` is the all-items element (the table's top-left corner) on a table control and on an ALV grid alike; on an ALV grid that pair is the only `tableCol` accepted, and any other rejects the selector. Either attribute may carry a selector variable |
| `colName` | on an ALV grid, the **technical** column name (`MATNR`), language-stable; on a classic `GuiTableControl`, the column's displayed title (`Depart.city`), in the logon language | alone, the column element (the grid's header, the table's column); with `tableRow`, the cell. The driver's default ALV cell is the grid's `id` + `tableRow` + `colName` |
| `colTooltip` | the column's tooltip (`Tooltip`, falling back to the accessibility tooltip, then the default quick-info), trimmed. On a table control and on an ALV grid alike it is **not** the header's displayed title: a column headed `G/L acct` has `colTooltip='G/L Account'`, one headed `Tax code` has `'Tax Code'` | with `tableRow`, a cell of either kind; alone, the column element of a table control only (an ALV grid rejects it without `tableRow` or `text`). The driver's default classic cell is the table's `id` + `tableRow` + `colTooltip`, a tooltip text in the logon language; `tableCol` is the language-independent form, which the driver itself writes where two columns share a tooltip. A title or tooltip two columns share matches the first of them |
| `relpath` | tree node path of node **display texts**, root-relative, slash-separated, a literal `/` escaped as `\/` (`Root/Child/Leaf`) | the value the Expand Tree activity takes |
| `itemId` | tree item inside a node: index (`0`…`n`) for list trees, column name for column trees | trees only |
| `idx` | position among matches | |

Which attributes a tag carries depends on the element it resolves to. `id`, or `leafid` in its place, finds the control; on a table control, ALV grid or tree every other attribute addresses an element inside the control. An attribute the element does not take fails the search; it is never skipped.

| Element | Its tag carries, besides `idx` |
|---|---|
| Table control, ALV grid or tree itself | `id` alone: `type='GuiGridView'` beside the grid's id finds nothing |
| Row, column, cell or all-items element of a table control or ALV grid | the control's `id` with only `tableRow`, `tableCol`, `colName`, `colTooltip`, `text`, in the combinations their rows above give |
| Tree node | the tree's `id` and `relpath` |
| Tree item | `itemId` alone, in a tag after its node's tag |
| Any other control: field, button, label, container | `id`, `text`, `tooltip`, `type` |

The attribute table is the whole `<sap>` schema. `name` (the technical field name), `labeledby` (the field's label), `iconName`, status and scroll attributes are readable through Get Attribute only: a selector that names one does not parse.

## Ids that hold and ids that shift

- **System toolbar** ids are fixed by function key: `tbar[0]/btn[11]` is Save on every screen that offers it. SAP GUI for Windows leaves a button the screen does not offer out of the tree, and a caption can change (`btn[15]` is Log Off on SAP Easy Access, Exit elsewhere).
- **Application toolbar** buttons and menu positions (`mbar/menu[4]/menu[2]`) shift between screens.
- **ALV grid** ids are unreliable across sessions; the driver's default grid cell uses the technical column name: `<sap id='<grid id>' tableRow='…' colName='…' />`.
- **Classic table cells** have a scripting id of their own (`…/tbl<TABLE>/<prefix><FIELD>[c,r]`) that resolves at run time, exact or as a regex (a slower tree walk). Its `[c,r]` count only the displayed rows and columns, and equal `tableCol='c' tableRow='r'` only while the table shows its first row and column.

## Classic lists

A classic list (an ALV list, a report's output) has no table element and takes no table attribute. Each cell is a `GuiLabel` `usr/lbl[<character column>,<line>]` and a row check box `usr/chk[<column>,<line>]`; the line counts the displayed list page, and the other cells of a row share its line. Only the displayed page is in the tree: first page `tbar[0]/btn[80]`, next page `tbar[0]/btn[82]`. A column's label pinned by its value is `<sap id='usr/lbl[<column>,*]' text='<value>' />`, the `*` staying inside its segment. The driver's default for a list label is its `text` plus `type`.
