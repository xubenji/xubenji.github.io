# vscode_1: 使用clangd格式化代码

------



------

### 1. 在vs code中安装clangd插件
    
在vscode搜索clangd插件并安装
安装好以后会和C/C++ IntelliSense冲突，所以要disable C/C++ IntelliSense（如果你安装了这个插件的情况下），disable如果不成功，使一下大写D，“Disable”。
当然，你也可以通过vs code的配置文件更改设置。
MacOS下，找到顶部view一栏，点击command Palette，再点击Preference:open user setting，再找到C_Cpp.intelliSenseEngine一栏设置为Disable。
这些设置完成以后我们需要开启保存时自动格式化代码选项。
找到"editor.formatOnSave"，改为true。并保存。

> 注：如果找不到vscode的设置文件，你可以尝试手动打开此文件，MacOS下vscode的配置文件在 /Users/xxxx(用户名)/Library/Application Support/Code/User/settings.json

### 2. 设置格式类型
clangd支持许多格式类型，Microsoft，Google，clang，clangd默认的格式为clang，我们要把他改成Microsoft类型的格式。
在vs code打开的项目文件夹下建立一个名为 “.clang-format“的文件夹。在文件夹中粘贴以下内容：

```
BasedOnStyle: Microsoft
AccessModifierOffset: -4
AlignConsecutiveMacros: true
AlignTrailingComments: true
AllowShortFunctionsOnASingleLine: Inline
AllowShortIfStatementsOnASingleLine: false
BreakBeforeBraces: Allman
ColumnLimit: 0
```
保存好以后重启vscode，再次进入工作目录，按住ctrl+s就可以发现你的代码被自动格式化为微软类型的格式了。
格式化后的代码展示：
```
static void thread_alloc(THREADID tid, CONTEXT *ctx, INT32 flags, VOID *v)
{
    /* store the old threads context */
    thread_ctx_t *tctx_prev = threads_ctx;

    /*
   * we need more thread contexts; optimized branch (not so frequent);
   *
   * NOTE: in case the tid is greater than tctx_ct + THREAD_CTX_BLK we
   * need to loop in order to allocate enough thread contexts
   */
    while (unlikely(tid >= tctx_ct))
    {
        /* reallocate space; optimized branch */
        if (unlikely((threads_ctx = (thread_ctx_t *)realloc(
                          threads_ctx, (tctx_ct + THREAD_CTX_BLK) *
                                           sizeof(thread_ctx_t))) == NULL))
        {
            /* failed; this is fatal we need to terminate */

            /* cleanup */
            free(tctx_prev);

            /* error message */
            fprintf(stderr, "%s:%s:%u\n", __FILE__, __func__, __LINE__);

            /* die */
            libdft_die();
        }

        /* success; patch the counter */
        tctx_ct += THREAD_CTX_BLK;
    }
}
```
这是格式化前的：
```
static void thread_alloc(THREADID tid, CONTEXT *ctx, INT32 flags, VOID *v) {
  /* store the old threads context */
  thread_ctx_t *tctx_prev = threads_ctx;

  /*
   * we need more thread contexts; optimized branch (not so frequent);
   *
   * NOTE: in case the tid is greater than tctx_ct + THREAD_CTX_BLK we
   * need to loop in order to allocate enough thread contexts
   */
  while (unlikely(tid >= tctx_ct)) {
    /* reallocate space; optimized branch */
    if (unlikely((threads_ctx = (thread_ctx_t *)realloc(
                      threads_ctx, (tctx_ct + THREAD_CTX_BLK) *
                                       sizeof(thread_ctx_t))) == NULL)) {
      /* failed; this is fatal we need to terminate */

      /* cleanup */
      free(tctx_prev);

      /* error message */
      fprintf(stderr, "%s:%s:%u\n", __FILE__, __func__, __LINE__);

      /* die */
      libdft_die();
    }

    /* success; patch the counter */
    tctx_ct += THREAD_CTX_BLK;
  }
}
```
明显Microsoft的要好看一些，当然你如果有喜欢的代码格式可以更改.format-clang文件。
