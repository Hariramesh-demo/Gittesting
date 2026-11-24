## Ctrl:

a) very reliable: automationid, role,
b) alright: labeledby, name, rowName
c) less reliable: aastate, virtualname, helpText
d) highly unreliable: text

`automationid` comes first unless it is generated (a number, a GUID) or repeats on every instance (list items, grid cells); then `name` and `role` identify the element.