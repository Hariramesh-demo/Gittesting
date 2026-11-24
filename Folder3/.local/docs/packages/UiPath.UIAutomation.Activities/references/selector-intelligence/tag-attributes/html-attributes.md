# `html` attributes

Browser tab tag. Part of [selector-reference.md](../selector-reference.md); reliability ranking: [html-instructions.md](../tag-names-instructions/html-instructions.md).

| Attribute | Holds | Notes |
|---|---|---|
| `app` | browser executable; selects the browser driver at run time | empty or missing is treated as Internet Explorer; value case-insensitive |
| `url` | page URL | value case-sensitive: `url='*Site*'` and `url='*site*'` are different selectors |
| `title` | the tab's page title | not the browser window's caption: no ` - Google Chrome` suffix |
| `htmlwindowname` | `window.name` | |
| `mobiledata` | mobile device binding | |
| `sapweb-ses-screen`, `sapweb-ses-transaction`, `sapweb-ses-client`, `sapweb-ses-user`, `sapweb-ses-program` | SAP Web GUI session context | |
| `ui5-ses-technicalName`, `ui5-ses-appId` | UI5 application context | |
| `idx` | position among matches | |
| `pid`, `tid`, `gid`, `bsid`, `excludehwnd`, `isremoteapp`, `useBrowserExtension` | runtime values | change with every run |

## `app` values

Values the driver recognises as a browser, compared case-insensitively:

| Browser | `app` |
|---|---|
| Google Chrome | `chrome.exe` |
| Microsoft Edge (Chromium) | `msedge.exe` |
| Mozilla Firefox | `firefox.exe` |
| Internet Explorer | `iexplore.exe`, also the default when `app` is empty or missing |
| Brave, Vivaldi | `brave.exe`, `vivaldi.exe` |
| Custom Chromium-based browser with the UiPath extension | `custom_browser` |
| Safari (macOS) | `Safari` |

Any other value is matched, with its wildcards or matching mode, against those browsers' process names. When none matches, the driver tries Internet Explorer, then a web view embedded in the process `app` names (a desktop application's web view), then a Chromium-based browser of that process name with the UiPath extension.
