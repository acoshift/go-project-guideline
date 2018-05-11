# VSCode

## Plugin

- Go
- ESLint
- markdownlint
- Sass
- Vetur
- Better TOML
- Beautify

## Config

```json
{
    "editor.tabCompletion": true,
    "editor.snippetSuggestions": "inline",
    "editor.insertSpaces": false,
    "editor.tabSize": 4,
    "editor.fontSize": 14,
    "editor.renderWhitespace": "none",
    "editor.rulers": [140],
    "editor.renderIndentGuides": false,
    "files.trimTrailingWhitespace": true,
    "explorer.confirmDragAndDrop": false,
    "go.liveErrors": {
        "enabled": true,
        "delay": 500
    },
    "files.exclude": {
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/vendor/**": true,
        "**/node_modules/**": true
    },
    "[go]": {
        "editor.formatOnSave": true
    },
    "files.associations": {
        "*.tmpl": "html"
    },
    "extensions.ignoreRecommendations": false,
    "emmet.triggerExpansionOnTab": true,
    "eslint.packageManager": "yarn",
    "go.addTags": {
        "tags": "json",
        "options": "json=",
        "promptForTags": false,
        "transform": "camelcase"
    },
    "html.format.endWithNewline": false,
    "html.format.wrapLineLength": 140,
    "html.suggest.angular1": false,
    "html.suggest.ionic": false,
    "html.validate.scripts": true,
    "html.validate.styles": true
}
```