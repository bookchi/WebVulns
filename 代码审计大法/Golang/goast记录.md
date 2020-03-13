## package ast


package ast声明了表示语法树的类型。

```go
// 表明src中是否有可导出的声明
func FileExports(src *File) bool
// 在适当的位置修剪Go源文件的AST，以便仅保留到处的节点：未导出的所有顶级标识符及其相关信息都被删除。删除非导出字段和类型的方法。file.Comments列表没有被改变。
```



```go
// 在过滤后，是否还有declared names
func FilterDecl(decl Decl, f Filter) bool
// timGo的AST——删除所有不通过过滤器f的名称（包括结构体的filed和接口的方法名称，但是不包括参数列表）
```



- node

  ```go
  type Node interface{
    Pos() token.Pos		// node的第一个字符的位置
    End() token.Pos		// 下一个node的第一个字符的位置
  }
  ```

  实现了Pos()，End()函数的，都是Node类型。

- expr

  ```go
  type Expr interface{
    Node
    exprNode()
  }
  ```

  实现了Pos(), End()，exprNode()函数的，都是Expr类型

- stmt

  ```go
  type Stmt interface{
    Node
    stmtNode()
  }
  ```

  实现了Pos(), End(), stmtNode()函数的，都是stmt类型，也是Node类型。

- spec

  ```go
  type Spec interface{
    Node
    specNode()
  }
  
  // 一个包的导入
  // io import "fmt"
  type ImportSpec struct{
    Doc			*CommentGroup		// 关联的文档；or nil
    Name		*Ident					// 本地package名称；or nil -- io
    Path		*Basiclit				// 导入的路径							-- fmt
    Comment	*CommentGroup		// line注释；or nil
    EndPos	token.Pos				// spec的结束
  }
  
  // 常量或变量声明
  type ValueSpec struct{
    Doc			*CommentGroup		// 关联的文档；or nil
    Names		[]*Ident				// 变量or常量的名字
    Type		Expr						// 类型；or nil（常量为nil
    Values  []Expr					// 初始化的value；or nil
    Comment	*CommentGroup		// line注释
  }
  
  // 类型声明
  type TypeSpec struct{
    Doc			*CommentGroup		// 关联的文档；or nil
    Name 		*Ident					// 类型的名称
    Assign	token.Pos				// 如果存在=，它的位置
    // *Ident, *ParenExpr, *SelectorExpr, *StarExpr, or any of the *XxxTypes
    Type		Expr						
    Comment	*CommentGroup		// line注释；or nil
  }
  ```

  

- decl

  ```go
  type Decl interface{
    Node
    declNode()
  }
  ```

  - baddecl

    ```go
    type BadDecl struct{
      From, To  token.Pos
    }
    ```

    

  - funcdecl

    ```go
    type FuncDecl struct{
      Doc		*CommentGroup
      Recv	*FieldList
      Name  *Ident
      Type  *FuncType
      Body. *BlockStmt
    }
    
    // functpye
    ```

  - Gendecl

    ```go
    type GenDecl struct{
      Doc		 *CommentGroup
      TokPos *token.Pos
      Tok		 token.Token
      Lparen []token.Pos
      Specs  []Spec
      Rparen token.Pos
    }
    ```

- exprstmt

  ```go
  type ExprStmt struct{
    X Expr
  }
  ```

- object

  ```go
  type Object struct{
    Kind ObjKind
    Name string				// 声明的名称
    Decl interface{}	// 对应field, xxxSpec, FuncDecl, LableStmt, AssginStmt, Scope;nil
    Data interface{}	// 对象的data；or nil
    Type interface{}	// type信息的占位符；may be nil
  }
  
  // 其中，objkind是int类型，描述了obj代表了什么(pkg, Typ, Var等)
  type ObjKind int
  const(
  	Bad ObjKind = iota	// 错误处理，0
    Pkg									// 就是package，1
    Con									// 常量，constant，2
    Typ									// 类型type，3
    Var									// 变量variable，4
    Fun									// 函数function，5
    Lbl									// 标签label，6
  )
  
  // 而各种类型
  type Package struct{
    Name		string							// 包名
    Scope		*Scope							// 包scope下所有的文件
    Imports map[string]*Object	// map：包id->包obj
    Files		map[string]*File		// map：filename->源文件
  }
  
  type Scope struct{	// Scope维护一组在其范围内声明的一组实体，以及a link-紧邻的外部范围
    Outer		*Scope
    Objects	map[string]*Object
  }
  
  type File struct{
    Doc				*CommentGroup		// 关联的文档；or nil   ？？？
    Package		token.Pos				// package关键字的位置
    Name			*Ident					// package name
    Decls			[]Decl					// 顶级声明；or nil -- 比如函数声明，变量常量声明
    Scope			*Scope					// 包作用域(this file only)
    Imports		[]*ImportSpec		// 文件中的imports
    Unresovle	[]*Ident				// 这个file中，未解析的标识符
    Comments	[]*CommentGroup	// 源文件的注释list	
  }
  ```

  
