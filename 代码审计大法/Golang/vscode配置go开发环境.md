# VS Code配置golang环境
一切从简，只说关键步骤。

## 基本配置

假设在windows环境下，先下载安装Go：[下载地址](https://golang.org/dl/)

然后在vscode中，安装搜索安装go插件。

Win下，按下`CTRL+SHIFT+P`，输入`go:install`,选择`Go:Install/Update Tools`,安装所有的插件
<img src="https://www.liwenzhou.com/images/Go/00_config_VSCode/15535665573387.jpg">

## 修改默认配置

有了基本配置后，发现还是不能自动导入包，参考其他博客做一下修改：

> 在 Preferences -> Setting 然后输入 go，然后选择 `setting.json`，填入你想要修改的配置

- 自动导入包

  ```json
  "go.autocompleteUnimportedPackages": true,
  ```

...

作者的json：

```json
{
  "go.goroot": "",
  "go.gopath": "",
  "go.inferGopath": true,
  "go.autocompleteUnimportedPackages": true,
  "go.gocodePackageLookupMode": "go",
  "go.gotoSymbol.includeImports": true,
  "go.useCodeSnippetsOnFunctionSuggest": true,
  "go.useCodeSnippetsOnFunctionSuggestWithoutType": true,
  "go.docsTool": "gogetdoc",
}
```

## Reference

- [VS Code 中的代码自动补全和自动导入包](<https://maiyang.me/post/2018-09-14-tips-vscode/>)
- [VS Code配置Go语言开发环境](<https://www.liwenzhou.com/posts/Go/00_go_in_vscode/>)

