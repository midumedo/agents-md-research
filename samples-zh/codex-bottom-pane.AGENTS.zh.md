# TUI 底部面板（状态机）

改动本目录下的 paste-burst 或 chat-composer 状态机时，文档要同步：

- 更新对应的模块文档（`chat_composer.rs` 和/或 `paste_burst.rs`），使其仍然是对当前行为的
  一份可读、自顶向下的说明。
- 实现与 docstring 保持一致，除非分歧是有意为之且已写明。

实用检查：

- 改完之后核一遍：文档只提到代码里真实存在的 API/行为（尤其是 Enter/换行路径与
  `disable_paste_burst` 的语义）。
