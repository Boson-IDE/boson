# Custom themes

Custom themes use Boson's unified theme registry. A single theme ID drives the IDE workbench, the editor's syntax colors and the integrated terminal.

## Create and manage

Open **Settings → Custom Themes** to:

- create a theme from the currently active palette;
- import or export a `.boson-theme.json` file;
- edit the full definition as JSON and validate it before saving;
- activate or delete a custom theme.

Custom themes are saved in Boson's local preferences and restored on the next launch. Themes contributed by [plugins](plugins.md) appear in the same picker.

## File format

```json
{
  "id": "custom:deep-space",
  "label": "Deep Space",
  "description": "A blue-black palette",
  "palette": {
    "appearance": "dark",
    "background": "#101827",
    "foreground": "#D8E4F5",
    "lineNumber": "#687994",
    "lineNumberActive": "#F4F7FC",
    "cursor": "#6DE2D0",
    "selection": "#263A55",
    "inactiveSelection": "#263A5599",
    "lineHighlight": "#17243A",
    "indent": "#24344D",
    "indentActive": "#3D5878",
    "border": "#24344D",
    "focus": "#75A7FF",
    "keyword": "#C792EA",
    "string": "#C3E88D",
    "function": "#82AAFF",
    "number": "#F78C6C",
    "comment": "#687994",
    "type": "#89DDFF",
    "variable": "#D8E4F5",
    "operator": "#89DDFF",
    "constant": "#FFCB6B",
    "delimiter": "#B8C7DB"
  }
}
```

## Rules

- All **22 palette entries** are required.
- Colors must be six-digit or eight-digit hexadecimal values (eight digits include alpha).
- `appearance` is `dark` or `light`.
- Imports are limited to 256 KiB.
- Theme files are data only and cannot execute code.
