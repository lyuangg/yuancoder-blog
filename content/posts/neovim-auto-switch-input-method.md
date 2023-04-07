---
title: "Neovim 自动切切换输入法脚本"
date: 2023-04-07T10:27:41+08:00
draft: false
tags:
- neovim
- vim
- lua
categories:
- 工具
---

<!--more-->

neovim 中输入法切换很繁琐， 写个脚本来自动切换。

```lua
-- 记录当前输入法
Current_input_method = vim.fn.system("/usr/local/bin/im-select")

-- 切换到英文输入法
function Switch_to_English_input_method()
    Current_input_method = vim.fn.system("/usr/local/bin/im-select")
    if Current_input_method ~= "com.apple.keylayout.ABC\n" then
        vim.fn.system("/usr/local/bin/im-select com.apple.keylayout.ABC")
    end
end

-- 设置输入法
function Set_input_method()
    if Current_input_method ~= "com.apple.keylayout.ABC\n" then
        vim.fn.system("/usr/local/bin/im-select " .. string.gsub(Current_input_method, "\n", ""))
    end
end

-- 进入normal模式时切换为英文输入法
vim.cmd([[
augroup input_method
  autocmd!
  autocmd InsertEnter * :lua Set_input_method()
  autocmd InsertLeave * :lua Switch_to_English_input_method()
augroup END
]])

```


