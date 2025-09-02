# Pacman

Pacman是Arch Linux及其衍生发行版中的软件包管理器，它负责安装、更新、删除和管理系统中的软件包。Pacman使用包数据库来跟踪已安装的软件包及其依赖关系，确保系统的一致性和稳定性。

## 安装软件

-   安装单个软件包：pacman -S 软件名
-   同时安装多个软件包：pacman -S 软件名1 软件名2 ...
-   安装软件，但跳过已经是最新版本的包：pacman -S --needed 软件名1 软件名2
-   显示操作信息后安装软件：pacman -Sv 软件名
-   仅下载软件包，不安装：pacman -Sw 软件名
-   安装本地软件包：pacman -U 软件名.pkg.tar.gz
-   安装远程软件包（非官方源）：pacman -U http://www.example.com/repo/example.pkg.tar.xz

## 更新系统

-   更新软件包数据库：`pacman -Sy`
-   升级所有已安装的软件包：`pacman -Su`
-   同时更新软件包数据库和升级所有包：`pacman -Syu`

## 查询系统

查询已安装的软件包（简单信息）：

```
pacman -Q
```

查询已安装的软件包（详细信息）：

```
pacman -Qs
```

查询从 Arch Linux 官方软件仓库安装的软件包：

```
pacman -Qe
```

查询某个软件包详细信息：

```
pacman -Qi 完整包名
```

## 卸载软件

-   仅卸载软件包，保留依赖：`pacman -R <软件名>`
-   卸载软件包，并显示详细信息：`pacman -Rv <软件名>`
-   卸载软件包及其不再需要的依赖：`pacman -Rs <软件名>`
-   卸载软件包及其所有依赖（慎用）：`pacman -Rsc <软件名>`
-   卸载软件包，删除不再被任何已安装软件包所需要的依赖：`pacman -Ru <软件名>`

## 清理缓存

清理所有缓存：

```
pacman -Sc
```

清理所有缓存，包括不再需要的软件包：

```
pacman -Scc
```

