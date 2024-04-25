

# Git Flow 工作流程

![LiangNing_2024-04-25_19-22-37](https://gitee.com/liangningi/typora_picture/raw/master/Go/202404252115938.png)

# 1. 项目初始化

创建`gitflow`文件夹，在其中创建`Readme.md`和`main.go`文件。文件内容如下：

`main.go`：

```go
package main

import "fmt"

func main() {
	fmt.Println("callmainfunction")
}
```

然后在`gitflow`文件夹中，进行`git`初始化：

```git
git init
git add .
git commit -m "first commit"
git tag v1.0 main
```



在`github`中，创建`gitflow`仓库。

然后建立本地与云端的联系：

```git
git remote add origin git@github.com:xxx/gitflow.git
```

其中：`git@github.com:xxx/gitflow.git`改为自己的创建的仓库即可。

然后将本地文件上传至云端

```git
git push origin main --tags
```

这样就完成了项目的初始化



# 2. 创建dev分支，并且创建`feature/order`分支

1. 基于`main`分支，创建一个常驻分支：`dev`

   ```git
   git checkout -b develop main
   ```

2. 基于 `develop` 分支，创建一个功能分支：`feature/print-hello-world`

   ```git
   git checkout -b feature/print-hello-world develop
   ```



# 3. 在`feature/order`分支开发

> 这里只开发了一半

在`feature/order`分支中，在`main.go`文件中开发功能，开发后的`main.go`文件如下：

```go
package main

import "fmt"

func main() {
	fmt.Println("callmainfunction")
    fmt.Println("Hello")
}
```



# 4. 紧急修复 BUG

我们正处在新功能的开发中（只完成了 `fmt.Println("Hello")`而非` fmt.Println("Hello World")`）突然线上代码发现了一个 Bug，我们要立即停止手上的工作，修复线上的 Bug，步骤如下。

```git
git stash # 1. 开发工作只开发了一半，还不想提交，可以临时保存修改至堆栈区
git checkout -b hotfix/print-error main # 2. 从 main 建立 hotfix 分支
```

3 修复 bug，callmainfunction -> call main function

```go
package main

import "fmt"

func main() {
	fmt.Println("call main function")
}
```

```git
git commit -a -m 'fix print message error bug' # 4. 提交修复
git checkout develop # 5. 切换到 develop 分支
git merge --no-ff hotfix/print-error # 6. 把 hoxfix 分支合并到 develop 分支
git checkout main # 7. 切换到 main 分支
git merge --no-ff hotfix/print-error # 8. 把 hoxfix 分支合并到 main 分支
git tag -a v1.1 -m "fix bug" # 9. 给 main 分支打 tag
git push origin main --tags # 10. 上传到云端
git branch -d hotfix/print-error # 11. 修复好后，删除 hotfix/xxx 分支
git checkout feature/print-hello-world # 12. 切换到开发分支下
git merge --no-ff develop # 13. 因为 develop 有更新，这里也同步更新一下
git stash pop # 14. 恢复到修复前的工作状态
```

在`git stash pop`时，会遇到冲突，进行解决即可：

```go
package main

import "fmt"

func main() {
<<<<<<< Updated upstream
        fmt.Println("call main function")
=======
        fmt.Println("callmainfunction")
        fmt.Println("Hello")
>>>>>>> Stashed changes
}

```

修改为：

```go
package main

import "fmt"

func main() {
        fmt.Println("callmainfunction")
        fmt.Println("Hello")
}
```




# 5. 继续开发

在`main.go`中继续开发：

```go
package main

import "fmt"

func main() {
	fmt.Println("callmainfunction")
    fmt.Println("Hello World")
}
```

# 6. 提交代码到`feature/print-hello-world`分支

```go
git commit -a -m "print 'hello world'"
```



# 7. 在`feature/print-hello/world`分支上做`code reiew`

首先，我们需要将 `feature/print-hello-world push` 到代码托管平台，例如 GitHub 上。

```go
git push origin feature/print-hello-world
```

然后，我们在 GitHub 上，基于 `feature/print-hello-world` 创建 `pull request`，如下图所示。

![image-20240425220712863](https://gitee.com/liangningi/typora_picture/raw/master/Go/202404252251754.png)

创建完 `pull request`之后，就可以指定 `Reviewers`进行`code review`，如下图所示：

![image-20240425224451219](https://gitee.com/liangningi/typora_picture/raw/master/Go/202404252244348.png)

# 8. code review 通过之后，由代码仓库 `matainer`将功能分支合并到 `develop`分支

```git
git checkout develop
git merge --no-ff feature/print-hello-world
```

# 9. 基于 `develop` 分支，创建 `release`分支，测试代码

```git
git checkout -b release/1.1 develop
go build -v . # 构建后，部署二进制文件，并测试
```

# 10. 测试失败

测试失败，因为我们要求打印“hello world”，但打印的是“Hello World”，修复的时候，

我们直接在 release/1.0.0 分支修改代码

`main.go`：

```go
package main

import "fmt"

func main() {
        fmt.Println("callmainfunction")
        fmt.Println("hello world")
}
```

修改完成后，提交并编译部署。

```git
git commit -a -m "fix hello world bug"
go build -v .
```

# 11. 测试通过后，将功能分支合并

将功能分支合并到 `master` 分支和 `develop` 分支。

```git
git checkout develop
git merge --no-ff release/1.1
git checkout main
git merge --no-ff release/1.1
git tag -a v2.0 -m "add print hello world" # 给main分支打标签
git push origin main --tags
```

# 12. 删除 `feature/print-hello-world`分支，也可以选择性删除`release/1.1分支`

```git
git branch -d feature/print-hello-world
```

