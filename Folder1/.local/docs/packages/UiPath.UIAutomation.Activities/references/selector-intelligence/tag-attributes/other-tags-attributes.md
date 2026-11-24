# `ctrl` and other tags

Part of [selector-reference.md](../selector-reference.md).

## `ctrl` — Active Accessibility element

Reliability ranking: [ctrl-instructions.md](../tag-names-instructions/ctrl-instructions.md).

Attributes: `automationid`, `role`, `labeledby`, `name`, `rowName`, `virtualname`, `helpText`, `accessibleDescription`, `text`, `aastate` (the element's current state), `idx` (position among matches).

## Other tags

| Tag | Attributes | Notes |
|---|---|---|
| `wpf` | `type`, `name`, `uid`, `header`, `automationid`, `content`, `text`, `tooltip`, `labeledby`, `groupname`, `row`, `col`, `rowName`, `colName`, `checked`, `isexpanded`, `idx`; driver internals `command`, `datastr`, `dotnethash`, `persistid`, `band*`, `auto*` | WPF normally surfaces through `uia` |
| `silverlight` | `role`, `name`, `text`, `idx` | Silverlight plug-in content only |
| `qt` | `name`, `role`, `tooltip`, `accessibleName`, `statusTip`, `text`, `rawtext`, `idx` | |
| `curl` | `cls`, `role`, `name`, `text`, `tableRow`, `tableCol`, `colName`, `idx` | Curl applets |
| `ax` | `role`, `subrole`, `name`, `title`, `identifier`, `filename`, `url`, `help`, `value`, `rowCount`, `columnCount`, `tableRow`, `tableCol`, `rowName`, `colName`, states, `idx` | macOS accessibility |
| `rdp`, `rdpapp` | the `<wnd>` attribute set ([wnd-attributes.md](wnd-attributes.md)) | remote runtime |
