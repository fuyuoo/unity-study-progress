# 模块 1 - 资源加载方式总览

## 学习目标

理解 Unity 四种资源加载方式的底层差异、适用场景、优缺点，能够在项目中做出正确的技术选型。

---

## 1. Resources

### 是什么
`Resources` 文件夹是 Unity 最早期的资源管理方式，放入该文件夹的资源会被**全部打包进游戏**，运行时通过路径加载。

### 加载方式
```csharp
// 同步加载
Texture2D tex = Resources.Load<Texture2D>("Textures/Hero");

// 异步加载
ResourceRequest req = Resources.LoadAsync<Texture2D>("Textures/Hero");
yield return req;
Texture2D tex = req.asset as Texture2D;
```

### 底层原理
- 所有 Resources 文件夹内容会被合并到一个**内部 Asset 索引表**
- 运行时索引表常驻内存，资源按需加载到内存
- 卸载：`Resources.UnloadAsset(asset)` 或 `Resources.UnloadUnusedAssets()`

### 缺点（为什么被抛弃）
1. **包体膨胀**：所有资源必须打进包，无法按需下载
2. **内存浪费**：索引表常驻，冷门资源也占用空间
3. **无法热更**：资源在包内，无法运行时替换
4. **无法精细控制**：没有引用计数，卸载困难

### 适用场景
- 极小型项目 / 原型
- 必须随包的少量资源（如启动界面、基础配置）

---

## 2. AssetBundle（AB）

### 是什么
AssetBundle 是 Unity 提供的**自定义资源打包格式**，将资源打成独立的 .bundle 文件，支持远程下载和按需加载。

### 加载方式
```csharp
// 从本地加载 AB
AssetBundle ab = AssetBundle.LoadFromFile(path);

// 从 AB 中加载资源
GameObject prefab = ab.LoadAsset<GameObject>("Hero");

// 卸载 AB（false=只卸载AB本身，true=同时卸载已加载的资源）
ab.Unload(false);
```

### 底层原理
- AB 文件 = Header（资源列表）+ Data（序列化资源数据）
- 加载时先读 Header 建立索引，再按需解压资源数据
- **依赖关系**：资源之间的引用（如材质引用纹理）会生成 Manifest 文件记录

### 核心问题：依赖管理
```
Hero.prefab → Material.mat → Texture.png
```
加载 Hero.prefab 时必须先加载依赖的 AB，否则资源丢失（粉红色）。

### 缺点
1. **手动管理依赖**：需要自己处理依赖加载顺序，容易出错
2. **引用计数需自己实现**：Unload 时机难以把握
3. **热更流程复杂**：版本对比、校验、下载需要自己写

### 适用场景
- 需要热更的项目（底层机制）
- 被 Addressables / YooAsset 封装后间接使用

---

## 3. Addressables

### 是什么
Addressables 是 Unity 官方基于 AssetBundle 的**上层封装**，通过"地址"（string key）加载资源，自动处理 AB 依赖和引用计数。

### 加载方式
```csharp
// 异步加载（推荐）
AsyncOperationHandle<GameObject> handle = Addressables.LoadAssetAsync<GameObject>("Hero");
handle.Completed += (op) => {
    GameObject go = Instantiate(op.Result);
};

// 释放（必须手动调用，否则内存泄漏）
Addressables.Release(handle);
```

### 底层原理
- 内部仍是 AssetBundle，但自动管理 AB 生命周期
- **引用计数**：每次 LoadAssetAsync +1，每次 Release -1，计数归零自动卸载 AB
- **Handle 机制**：每次加载返回 Handle，Release Handle 才减引用计数

### 核心陷阱
```csharp
// 错误：忘记 Release，导致内存泄漏
var handle = Addressables.LoadAssetAsync<Texture2D>("icon");
// ... 用完后没有 Release

// 正确：用完必须 Release
Addressables.Release(handle);
```

### 适用场景
- Unity 官方推荐的现代资源管理方式
- 中大型项目，需要热更但不想手动管理 AB

---

## 4. YooAsset

### 是什么
YooAsset 是国内团队开发的**开源资源管理框架**，同样基于 AssetBundle，但提供比 Addressables 更灵活的热更流程控制。

### 核心优势
1. **热更流程可控**：版本对比、断点续传、强更/弱更逻辑全部暴露给开发者
2. **分组策略**：按模块、按场景灵活分组打包
3. **引用计数封装**：比原生 AB 安全，比 Addressables 更透明
4. **编辑器模拟模式**：开发期不需要打 AB，直接从 AssetDatabase 加载

### 加载方式
```csharp
// 初始化
var package = YooAssets.CreatePackage("DefaultPackage");
YooAssets.SetDefaultPackage(package);

// 加载资源
var handle = package.LoadAssetAsync<GameObject>("Assets/Prefabs/Hero.prefab");
yield return handle;
GameObject go = handle.AssetObject as GameObject;

// 释放
handle.Release();
```

### 适用场景
- 需要精细控制热更流程的项目
- 国内手游项目主流选择

---

## 四种方式横向对比

| 特性 | Resources | AssetBundle | Addressables | YooAsset |
|------|-----------|-------------|--------------|----------|
| 热更支持 | ❌ | ✅ | ✅ | ✅ |
| 依赖管理 | 自动 | 手动 | 自动 | 自动 |
| 引用计数 | 无 | 手动 | 自动 | 自动 |
| 热更流程控制 | - | 手动 | 半自动 | 全可控 |
| 学习成本 | 低 | 高 | 中 | 中高 |
| 适合项目规模 | 极小 | 底层/封装后 | 中大型 | 中大型 |

---

## 关键问题自测

1. `Resources.UnloadAsset` 和 `Resources.UnloadUnusedAssets` 有什么区别？
2. AssetBundle 的 `Unload(true)` 和 `Unload(false)` 分别会发生什么？
3. Addressables 的 Handle 如果不 Release 会怎样？
4. 为什么 Resources 方式会导致包体膨胀？

---

## 参考资料

- [Unity 官方 - AssetBundle 文档](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)
- [Unity 官方 - Addressables 文档](https://docs.unity3d.com/Packages/com.unity.addressables@latest)
- [YooAsset 官方文档](https://www.yooasset.com/)
- [Unity 官方博客 - 资源管理最佳实践](https://blog.unity.com/technology/assets-best-practices-series)
