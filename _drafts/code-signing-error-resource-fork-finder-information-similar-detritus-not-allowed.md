Technical Q&A QA1940

# Code signing fails with error 'resource fork, Finder information, or similar detritus not allowed'

这是一个安全强化更新，在iOS 10，macOS Sierra，watchOS 3和tvOS 10引入。代码签名不再允许应用程序包中的任何文件具有包含资源分支或Finder信息的扩展属性。

可以通过运行如下的命令来查看哪个文件引起的这个错误，示例：

```
$ xattr -lr Foo.app
```

也可以使用`xattr`命令移除所有的扩展属性：

```
$ xattr -cr <path_to_app_bundle>
```

注意，使用Finder的Show Package Contents命令浏览文件包中的文件可能会导致Finder信息被添加到这些文件。 否则，请审核构建过程以查看要添加扩展属性的位置。