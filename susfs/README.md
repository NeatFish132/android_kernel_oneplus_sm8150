# SUSFS v2.3.0 — SM8150 / Linux 4.14.357 内核钩子补丁

`50_add_susfs_in_kernel-4.14-sm8150.patch` 只供 `.github/workflows/build-baka.yml` 在 `susfs=true` 时应用到 `17-susfs` 分支源码。仓库里的源码本身保持 Stage A（Manual Hook）状态不变。

## 来源与适配

- 起点是 [souta-lab/agate-bakasu-susfs](https://github.com/souta-lab/agate-bakasu-susfs) 的 4.14 移植（`bb08f9bd..a5b8168`）；该移植以 SUSFS v2.3.0 non-GKI 4.9 port 和上游 GKI 5.10 补丁为基础。
- 上游 [susfs4ksu gki-android16-6.12](https://gitlab.com/simonpunk/susfs4ksu/-/tree/gki-android16-6.12) 的补丁在本树上有 31/127 个 hunk 失败，而且依赖 `handle_inode_event`、`fs/proc/bootconfig.c` 等 4.14 中不存在的接口，所以只用作版本对照。
- 针对本树做的修改：
  - `fs/namespace.c`：本树已经 backport 了 `ida_alloc*()`，因此 SUS_MOUNT 的 mnt_id/group_id 分配改用 `ida_alloc_min()`，并在 `mnt_free_id()` 中跳过 unshared ksu mount。
  - `fs/open.c`、`fs/stat.c`：删除 Manual Hook 版本的钩子，换成 SUSFS 版本（使用 `struct filename **` 签名）。
  - `fs/exec.c`、`kernel/reboot.c`：守卫宏从 `CONFIG_KSU_MANUAL_HOOK` 改为 `CONFIG_KSU`。
  - 新增 `kernel/sys.c` 中的 `ksu_handle_setresuid`、`fs/read_write.c` 中的 `ksu_handle_sys_read`（BakaSU 三参数签名）、`drivers/input/input.c` 中的 `ksu_handle_input_handle_event`。Stage A 由 BakaSU 的 Auto Hook 处理这些点，而 SUSFS 模式需要显式钩子。

## 已验证

- 补丁可以用 `git apply --check` 零 fuzz 应用到 `17-susfs`（`44c9ead`）。
- 用 clang 21 + LTO 编译了 21 个改动文件、`fs/susfs.o` 和 BakaSU `d5e9e025`，0 错误、0 警告。
- BakaSU `inline_hook_check` 的 7 个钩子检查全部为 `found`，识别结果为 `SUSFS_VERSION: v2.3.0`。
- 核心、BakaSU 和钩子之间的 `susfs_*`/`ksu_*` 符号交叉引用全部可以解析。
- 尚未验证：完整 vmlinux 链接（由 CI 的 proton-clang 完成）以及实机启动。

## 许可证
