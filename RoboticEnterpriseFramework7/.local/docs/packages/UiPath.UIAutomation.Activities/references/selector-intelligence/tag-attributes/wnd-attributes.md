# `wnd` attributes

Window tag: top level for every desktop technology and for SAP GUI; nested, a Win32 child window that owns a handle (`<wnd cls='…' ctrlid='…' ctrlname='…' />`). Part of [selector-reference.md](../selector-reference.md); reliability ranking: [wnd-instructions.md](../tag-names-instructions/wnd-instructions.md).

| Attribute | Holds | Notes |
|---|---|---|
| `app` | executable name (`saplogon.exe`) | required on a top-level `wnd`; value case-insensitive |
| `cls` | window class (`SAP_FRONTEND_SESSION`, `#32770`, `Chrome_WidgetWin_1`) | |
| `title` | window caption | |
| `aaname` | accessible name | |
| `ctrlname` | Windows Forms control name | on a Windows Forms form, its name; `automationid` carries the same value |
| `ctrlid` | Win32 control id | stable in fixed dialogs; positional in dynamic UIs |
| `automationid` | UI Automation id of the window | |
| `idx` | position among matches | |
| `aastate` | accessibility state | current state |
| `pid`, `tid`, `gid`, `appid`, `isremoteapp`, `hwnd` | runtime identities | change with every run |

SAP GUI session attributes on `wnd`: [sap-attributes.md § Session window](sap-attributes.md#session-window).

## How a window selector filters

- `app` and `cls` exclude every window they do not match.
- Under a `ByInstance` Application Card, `title` ranks the instance's windows and excludes none. A window selector without the title leaves the windows of one process and class tied, and a target resolves in whichever of them first holds a match.
- A scope selector that matches several windows of the application at once leaves the target's window undecided.
- A window tag at the head of an element selector that has a scope selector is searched for inside the scope's window and matches nothing. With no scope selector it is the window selector.
