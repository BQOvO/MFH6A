# 上游模板改动记录 / Patches over the generated scaffold

`create-maa-project --update` 会按模板重新渲染下面这些 managed file，本仓库的改动会被无提示地打回。
升级模板之后请按这份清单重新应用，或者先 diff 一遍。

| 文件                                     | 改了什么                                                                                                           | 为什么                                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| `tools/build-release.mjs`                | 新增 `releaseArtifactName = "MFH6A"`；`guiEntrypointName()` 全平台用它：Windows 返回 `MFH6A.exe`，其余返回 `MFH6A` | 包里的 GUI 可执行文件要用显示名，不能是小写 slug                                             |
| `tools/build-release.mjs`                | `findUnixExecutableFiles()` 的名单里加入 `releaseArtifactName`                                                     | Unix 包的可执行位检查和归档断言都按这个集合挑文件，漏了会让 tar 里的入口丢权限而检查静默通过 |
| `tools/macos-app-bundle.mjs`             | `executable` 默认值 `mfh6a` → `MFH6A`                                                                              | 打包后按这个名字在 `.app` 里找入口并生成 `CFBundleExecutable`                                |
| `.github/workflows/release.yml`          | Unix 归档的 `awk` 白名单加入 `MFH6A`                                                                               | 该白名单区分大小写，不改则入口不在检查范围，permission 缺失会被放过                          |
| `.github/workflows/package-smoke.yml`    | 两处 `./mfh6a` → `./MFH6A`（`test -x` 与启动探针）                                                                 | 和上面同步，否则 macOS 冒烟找不到文件                                                        |
| `.github/workflows/release.yml`          | git-cliff 找不到上一个稳定 tag 时回退到全历史，而不是 `exit 1`                                                     | 首次发版（尤其第一个 tag 就是 `-beta`）没有稳定 tag，原逻辑会让 release job 连不上           |
| `maa-project.json`                       | `runtime.mfa.enabled` 改为 `false`                                                                                 | 只发 MXU 一种 GUI 包，MFAA 那套不打包也不下载 runtime                                        |
| `tools/build-release.mjs`、`release.yml` | 只有一个 GUI 启用时，产物名不带 `-MFAA`/`-MXU` 后缀（`MFH6A-<os>-<arch>-<tag>.zip`）                               | 单 GUI 时后缀没有区分作用，只会让文件名变长                                                  |

不能一起改名的地方，改之前先想清楚：

- `interface.json` 的 `name`、`package.json` 的 `name`、`maa-project.json` 的 `project.slug` 必须是 `mfh6a`。
  `tools/build-release.mjs` 里有 `if (interfaceJson.name !== projectSlug) throw`，npm 也要求包名小写。
- `mfh6a.icns` 保持小写，由 `displayName.toLowerCase()` 推导，`package-smoke.yml` 里断言的就是它。
