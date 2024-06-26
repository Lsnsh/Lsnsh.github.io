---
title: Git 提交消息格式和历史记录维护
title_en: 2024-06-25-git-commit-message-and-history-maintenance
date: 2024-06-25 14:35:30
tags:
  - Git
  - Guide
---

## Introduction

Git 提交消息格式的规范，接触到的总体来说主要就两种，第一种是不带类型前缀，大写字母开头，通常会是动词，常见的开头像是 `Add, Fix, Modify, Update` 这样的。

```log
- Add script to script loader when strategy prop is undefined (#65585)
- Update font data (#65768)
```

至于 [Git Commit Message With Emoji 😜][gitmoji] ，则可以说是第一种方式的变体，更加风格化。

第二种是带前缀类型，常见的前缀像是 `feat, fix, docs, refactor` 这样的，以冒号分隔提交类型和描述，描述以小写字母开头，后续内容将主要围绕第二种展开。

```log
- fix: ensure websockets are correctly passed (#65759)
- feat: remove font family hashing in next/font css (#53608)
- (tag: v14.3.0-canary.63) v14.3.0-canary.63
- fix(next): add alias to new react exports (#65761)
```

上述两段提交记录，都是摘自 [Next.js Commits][nextjs]，是的，你没看错，两种提交消息格式的都有。

## Git Commit Message Header format

> **Commit Message Format**
>
> _This specification is inspired by and supersedes the [AngularJS commit message format][commit-message-format]._
>
> We have very precise rules over how our Git commit messages must be formatted.
> This format leads to **easier to read commit history**.
>
> --- [Angular CONTRIBUTING][angular-contributing-commit]

`git commit` 时，书写清晰的提交信息，能帮助理解每次改动的目的和影响范围。

```
<type>(<scope>): <short summary>
  │       │             │
  │       │             └─⫸ 提交改动的简短总结或描述，改动的内容、起因等
  │       │
  │       └─⫸ 提交改动的影响范围: 页面名称|模块名称|组件名称｜或者 1-2 级能直观表述改动范围的内容
  │
  └─⫸ 提交改动的类型: build|ci|docs|feat|fix|perf|refactor|test|chore|...
```

`<type>` 和 `<summary>` 字段是必填, `(<scope>)` 字段是可选的。

#### Type

提交改动的类型：

- **build**: 影响构建系统或外部依赖关系的改动
- **ci**: CI 配置文件和脚本的改动
- **docs**: 文档改动
- **feat**: 功能新增
- **fix**: 问题修复
- **perf**: 提高性能的改动
- **refactor**: 既不修复问题也不添加功能的改动
- **test**: 添加测试或更新现有测试代码的改动
- **style**: 代码格式或页面样式风格的改动
- **chore**: 一些琐碎的事或小改动
- **revert**: 撤销旧的提交所做的改动
- **BREAKING CHANGE**: 破坏性的改动
- **DEPRECATED**: 弃用的内容
- ...

#### Scope

提交改动的影响范围：

- **页面名称**
- **模块名称**
- **组件名称**
- 或者 1 ～ 2 级能直观表述改动范围的内容

#### Summary

提交改动的简短总结或描述，改动的内容、起因等：

- 陈述句
- 首字母通常不大写
- 末尾没有句号

## Git Commit History Maintenance

`git` 操作时，以尽量维护相对线性的提交历史为目标，能帮助回顾历史和一些自动化工作。

#### Merge and Rebase

```bash
git merge
git merge --no-ff

git rebase <branch name>
git rebase --abort
git rebase --continue
```

`git merge` 保留了分支的历史和合并点，而 `git rebase` 则创建一个更线性的历史。

在个人开发分支上工作时，使用 `git rebase` 可以保持提交历史的简洁；在公共分支或需要保留详细合并信息的情况下，可以使用 `git merge`。

两者在处理冲突时都可能需要手动干预，但 `git rebase` 可能需要在多个提交上重复解决相同的冲突，考虑到处理冲突的繁琐程序，可以随时通过 `git rebase --abort` 中断，改为 `git merge`。

#### Reset and Revert

```bash
git reset
git reset --hard
git reset HEAD^
git reset <commit hash>

git revert <commit hash>
```

`git reset` 会改变分支的历史，而 `git revert` 通过添加新的提交来撤销更改，保持历史不变。

在个人开发分支或者确定没有其他人基于你的更改工作时，可以使用 `git reset`。在公共分支上，使用 `git revert` 以避免影响其他协作者。

#### Cherry-Pick

```bash
git cherry-pick <commit hash>
```

`git cherry-pick` 命令用于将一个或多个其他分支上的提交应用到当前分支。这个命令允许你选择性地选择某些提交并将它们作为新的提交引入到当前分支中。

#### Push With Force

```bash
git push --force
```

`git rebase` 或 `git reset` 时，会改变分支的历史，为了更新远程分支，免不了需要强制推送。

在个人开发分支上工作时，使用 `git push --force` 前需确保无误，无法确定时，建议先对分支进行备份。在公共分支上，应避免使用强制推送，除非明确知道自己在做什么。

## Reference Links

- [https://github.com/carloscuesta/gitmoji][gitmoji]
- [https://github.com/vercel/next.js/commits/canary/][nextjs]
- [https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit][angular-contributing-commit]
- [https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#][commit-message-format]
- [https://www.conventionalcommits.org/zh-hans/][conventionalcommits]
- [https://git-scm.com/docs][git-docs]

[gitmoji]: https://github.com/carloscuesta/gitmoji
[nextjs]: https://github.com/vercel/next.js/commits/canary/
[angular-contributing-commit]: https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit
[commit-message-format]: https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#
[conventionalcommits]: https://www.conventionalcommits.org/zh-hans/
[git-docs]: https://git-scm.com/docs
