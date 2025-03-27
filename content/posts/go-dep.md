+++
date = '2023-02-01T00:00:00+08:00'
title = 'Go依赖管理'
tags = ["Go"]
+++
最初起源于Google公司内部的代码依赖管理方式：所有源代码放在一个巨大的仓库中，直接使用路径引用，无包版本控制，无第三方仓库，从当前分支中构建所有内容。
- 添加依赖意味着将仓库代码克隆到本地`$GOPATH`中缓存，版本指向克隆时的主分支。

2017年Russ Cox 在GopherCon发表[Toward Go2](https://go.dev/blog/toward-go2)后，Go1.11发布，兼容性作为Go语言演化的第一考虑。一个模块是一组相关的Go包集合，被当做一个独立的单元进行统一版本管理，为了`可复制构建`和`新旧兼容`。
- `1.11`版本引入Go Modules，保持与Go1兼容，默认为`GO111MODULE=off`
- `1.13`版本默认`GO111MODULE=auto` 检测是否包含go.mod，如果有则启用module模式
- `1.14`版本GoMod可供生产使用，鼓励用户从其他以来管理系统迁移到Go Mod
- `1.16`版本默认`GO111MODULE=on`

# 版本号
默认版本号结构为`v<major>.<minor>.<patch>[-meta]` 参考[Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)
- `Major` 发布不兼容版本时递增，且Minor, Patch置为0
- `Minor` 发布向后兼容的更改时递增，且Patch置为0
- `Patch` 公共接口不做修改，仅仅是Bug修复或部分优化时递增
- `-pre` 预发版，如`v1.2.3`之前发布`v1.2.3-pre`
- `+incompatible` 构建元数据中保留的版本，标识主版本号升级到2以上（第一个非兼容版本）时使用。参考 [Compatibility with non-module](https://go.dev/ref/mod#non-module-compat)

## 伪版本
通过时间戳/提交哈希，固化一个非稳定版本(如`0.x.x`, `-pre`) 参考 [Pseudo Version](https://go.dev/ref/mod#glos-pseudo-version)。慎重依赖非稳定版本，避免重复构建失效。
```
vx.y.z-yyyymmddhhmmss-abcdef
```

## 版本路径
当模块出现一次非兼容升级时，路径必须有一个类似`/v2`的后缀。以遵循`同路径必须兼容`原则。
```
example.com/mod => exmaple.com/mod/v2
```
## 版本选择
- 如果go.mod中已标记版本号，使用`固定版本`
- 未指定版本，`import`时导入`最新版本`
- 某个模块被多个模块间接依赖，使用该模块的`最小可用版本`
```
M v1.0.0 -> module A
                      module X(M v1.1.1) // 确保A/B模块同时兼容
M v1.1.1 -> module B
```

# 包管理
- `go mod init <module>` 初始化模块
- `go get <module>` 添加、升级最新版
- `go get <module>@<version>` 添加、升级、降级指定版
- `go get <module>@none` 删除模块，并将需要它的模块降级
- `go get -u <module>` 升级到minor, patch到最新可用版本
- `go mod tidy` 自动清理未使用的模块
- `go mod graph` 模块依赖信息
- `go mod why <module>` 模块依赖路径
- `go list -m all` 列出全部模块

# 参考
- [Go & Versioning](https://research.swtch.com/vgo)
- [The Principles of Versioning in Go](https://research.swtch.com/vgo-principles) by `Russ Cox`, Go团队Leader
