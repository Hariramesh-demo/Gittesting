# `java` attributes

Java Swing/AWT element tag. Part of [selector-reference.md](../selector-reference.md); reliability ranking: [java-instructions.md](../tag-names-instructions/java-instructions.md).

| Attribute | Holds | Notes |
|---|---|---|
| `role` | accessible role (`push button`, `text`, `combo box`, `check box`, `radio button`, `toggle button`, `list`, `list item`, `table`, `tree`, `label`, `menu item`, `page tab`, `frame`, `dialog`, `panel`) | matched case-insensitively |
| `cls` | simple class name (`JButton`, `BusyComboBox`, `JideChart$1`) | |
| `accessibleClass`, `hasTableAncestor` | | |
| `name` | accessible name (caption) | |
| `rowName`, `colName`, `tableRow`, `tableCol` | table headers and position | |
| `virtualname` | the Swing component name (`Component.getName()`) when the component has one and its accessible name is set or nothing else names it. Otherwise the virtual accessible name: accessible name, else description, else, for a text, password text, combo box, list, table, spin box, slider or date editor not directly under a table or tool bar, the label that labels it, a label left of or above it in the same parent, or (text, combo box) a button left of it. A label's own text; a check box's value `true` / `false`; empty on a check box or combo box cell of a table | three different things under one name: an identifier where it holds a component name, a neighbouring label's text (which repeats across fields labelled alike), or a check box's current value |
| `javastate`, `backgroundColor`, `foregroundColor` | | |
| `idx` | position among matches | |

The driver builds its default `java` selector from `role`, `name`, `cls`, the table attributes and `idx`, never from `virtualname`, which it lists only among the element's other attributes. A default that ends in `idx` therefore says nothing about `virtualname`: read it in the attribute listing.
