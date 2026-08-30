# 改 Mod 工程 — BedWars1058 可视化商店编辑器

基于开源小游戏插件 [BedWars1058](https://github.com/andrei1058/BedWars1058) 的二次开发工程。
在保留原插件全部功能的基础上，**新增了一个游戏内可视化商店编辑器**，让服主无需手动改
`shop.yml`，直接通过 GUI 增删改商店分类、商品和 tier，并支持把背包里带完整 NBT 的
（模组）物品拖进商店。

> 适用服务器：Arclight 1.20.1（Forge 47.4.18）。模组物品的 NBT 会以 base64 序列化写入
> `shop.yml`，实现与纯 Bukkit/Spigot 物品一致的商品化。

---

## 环境要求

| 项目 | 要求 |
| --- | --- |
| 服务器核心 | Arclight 1.20.1（Forge 47.4.18） |
| Java | 11 或更高 |
| 构建工具 | Maven |

> 源码 `pom.xml` 版本基线为 `25.2`；当前服务器实际部署的插件版本为 `25.5-SNAPSHOT`。
> 补丁是在 `25.5-SNAPSHOT` 的 jar 基础上直接替换 class 得到的（见「补丁与部署」）。

---

## 主要改动（魔改内容）

### 1. 可视化商店编辑器（新增）

- 命令 `/bw shopeditor` 打开编辑器主菜单，权限节点 `bw.shopeditor`。
- 支持**分类**、**商品（content）**、**tier** 三层结构的增删改。
- 支持从玩家背包**拖拽物品**进编辑器，物品的完整 NBT（含模组物品）会被序列化保存。
- 支持设置价格、货币、永久物品（permanent）、不可破坏（unbreakable）等属性。
- 编辑器内的改动可即时保存（`shop.yml`）。

### 2. 稳定性修复

- 修复「新建分类缺少 `category-content` 节点」导致插件启动时
  `ShopCategory` 构造 NPE、进而拖垮整个 `onEnable` 的问题。
- 分类名 `sanitize` 逻辑拦截纯符号 / 中文名，避免生成非法节点名。
- `ShopCategory` 构造器增加空值防护：缺少 `category-content` 时告警并跳过，
  不再让单个异常数据把整个服务器带崩。

---

## 改动涉及的主要文件

新增：

- `bedwars-plugin/src/main/java/com/andrei1058/bedwars/shop/editor/ShopEditor.java`
- `bedwars-plugin/src/main/java/com/andrei1058/bedwars/shop/editor/ShopEditorListener.java`
- `bedwars-plugin/src/main/java/com/andrei1058/bedwars/commands/bedwars/subcmds/sensitive/CmdShopEditor.java`

修改：

- `.../configuration/Permissions.java` — 新增 `PERMISSION_SHOP_EDITOR` 权限。
- `.../commands/bedwars/MainCommand.java` — 注册 `shopeditor` 子命令。
- `.../shop/ShopManager.java` — 注册编辑器监听器。
- `.../shop/main/ShopCategory.java` — 增加 `category-content` 空值防护。
- `bedwars-api/.../api/configuration/ConfigPath.java` — 新增 `SHOP_CATEGORY_ITEM_NBT` 等常量
  （支持 base64 序列化的模组物品）。

---

## 目录结构

```
改mod工程项目/
├── BedWars1058-25.9/          # 插件源码（Maven 多模块工程）
│   ├── bedwars-plugin/        # 主模块（含商店编辑器）
│   ├── bedwars-api/           # API 模块
│   ├── versionsupport_*/      # 各版本适配（1.8 ~ 1.20.4）
│   └── resetadapter_*/        # 地图重置适配器
├── server/                    # 运行中的服务器（Arclight 1.20.1）
│   └── plugins/BedWars1058/   # 插件数据目录（shop.yml 等）
├── bedwars-plugin-25.5-SNAPSHOT.jar          # 原始备份
├── bedwars-plugin-25.5-modshop-patched.jar   # 补丁第 1 版
├── bedwars-plugin-25.5-modshop-patched2.jar  # 补丁第 2 版
└── bedwars-plugin-25.5-modshop-patched3.jar  # 补丁第 3 版（最新）
```

---

## 构建

在 `BedWars1058-25.9/` 目录下执行：

```bash
mvn clean package -pl bedwars-plugin -am
```

- `-pl bedwars-plugin`：只构建主模块。
- `-am`：同时构建它依赖的模块（api、versionsupport 等）。

产物位于 `bedwars-plugin/target/`，jar 名形如 `bedwars-plugin-<version>.jar`
（源码基线版本为 25.2，若需与线上 25.5-SNAPSHOT 对齐，请先调整 `pom.xml` 版本号）。

---

## 补丁与部署

线上运行的 `bedwars-plugin-25.5-SNAPSHOT.jar` 是 25.5-SNAPSHOT 版本，源码基线为 25.2。
补丁流程为：把源码改动编译出的 class，替换进 25.5-SNAPSHOT 的 jar 内。

当前最新补丁为 `bedwars-plugin-25.5-modshop-patched3.jar`，部署方式：

```bash
# 备份线上 jar
copy server\plugins\bedwars-plugin-25.5-SNAPSHOT.jar server\plugins\bedwars-plugin-25.5-SNAPSHOT.jar.bak

# 用最新补丁覆盖
copy bedwars-plugin-25.5-modshop-patched3.jar server\plugins\bedwars-plugin-25.5-SNAPSHOT.jar
```

部署后**重启服务器**，确认控制台无 `Error occurred while enabling BedWars1058` 报错。

---

## 使用说明

1. 拥有 `bw.shopeditor` 权限的玩家执行 `/bw shopeditor`。
2. 在 GUI 中：
   - 新增 / 编辑 / 删除**分类**；
   - 在分类下新增 / 编辑 / 删除**商品**与 **tier**；
   - 点击图标槽后，把背包里的物品（含模组物品）放上去即可替换为商品。
3. 改动后点击「保存」，配置写入 `server/plugins/BedWars1058/shop.yml`。

> 分类名请使用英文、数字、下划线或连字符；纯符号 / 中文名会被拦截。

---

## 许可证

本项目基于 [BedWars1058](https://github.com/andrei1058/BedWars1058)（Andrei Dascălu），
遵循 GNU GPL v3.0 协议。详情见 `BedWars1058-25.9/LICENSE`。
