title: Sublime Text 3 个人配置
date: 2015-12-23 22:47:42
tags: [Sublime]
---

Sublime Text 3 个人配置。

- [package control](https://packagecontrol.io/installation)
- [Emmet](https://github.com/sergeche/emmet-sublime#readme)
- [material-theme](https://github.com/equinusocio/material-theme)

个人配置:


    {
    	"color_scheme": "Packages/Theme - Spacegray/base16-eighties.dark.tmTheme",
    	"font_size": 14,
    	"ignored_packages":
    	[
    		"Vintage"
    	],
    	"markdown_binary_map":
    	{
    		"multimarkdown":
    		[
    			"/usr/local/bin/multimarkdown"
    		]
    	},
    	"theme": "Material-Theme.sublime-theme",
    	"color_scheme": "Packages/Material Theme/schemes/Material-Theme.tmTheme",
    	"hot_exit": true,
    	"remember_open_files": true,
    
    	"translate_tabs_to_spaces": true,
        "tab_size": 4
    }

**Mac下点击关闭窗口(红点)相当于关闭文件，因此需要用`cmd + q`关闭Sublime
，下次打开时才会保存上次打开的文件。囧**



