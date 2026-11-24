## Uia: 

a) very reliable: automationid, role,
b) alright: cls, name, helpText, legacyAccHelp
c) specific for tables: tableCol, tableRow
d) last resort, only use if needed to make the selector unique: roleint, enabled, ispwd, kbfocus

`automationid` comes first unless it is generated (a number, a GUID) or repeats on every instance (list items, grid cells); then `name` and `role` identify the element.