---
title: sublime插件整理
tags: [sbulime,插件]
---
# sass
# less
# babel
# emmet
* emmet jsx
// add to Preferences > Key Bindings - User
// see http://stackoverflow.com/a/26619524 for context
```
{ "keys": ["tab"], "command": "expand_abbreviation_by_tab",
  "context": [
    {
      "operand": "source.js", 
      "operator": "equal", 
      "match_all": true, 
      "key": "selector"
    },
    {   
      "key": "selection_empty", 
      "operator": "equal", 
      "operand": true,
      "match_all": true 
    }
  ]
},
{ "keys": ["tab"], "command": "next_field", "context":
  [
    { "key": "has_next_field", "operator": "equal", "operand": true }
  ]
}
```
sublime 显示空格和制表符
Preferences->Settings-User
"draw_white_space": "all"

# 符号高亮
* Bracket​Highlighter

# 扩展右键菜单
* SideBarEnhancements

# 语法检查
* Sublime​Linter
* SublimeLinter-contrib-eslint
* ESLint-Formatter

# js提示
* JavaScript Completions

# 自动提示文件名
* AutoFileName

# 文件图标
* A File Icon

# markdown 预览
* Markdown Extended

# 不同的编辑器之间保持一致的编码风格
* EditorConfig