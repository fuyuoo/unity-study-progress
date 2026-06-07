# Unity 学习系统 - 行为规范

## 核心规则

- 全程使用中文
- 每次对话开始时，**必须先读取 `progress.md`** 确认当前模块和状态，再做任何事

## 文件结构

| 文件 | 用途 |
|------|------|
| `progress.md` | 当前学习进度，模块状态 |
| `wrong_answers.md` | 错题本，含原理说明 |
| `module{N}_*.md` | 各模块学习资料 |
| `study.md` | 原始阶段规划（只读参考） |

GitHub 仓库：https://github.com/fuyuoo/unity-study-progress

---

## 学习流程

### 用户说"开始学习"或"继续学习"
1. 读取 `progress.md`，找到第一个状态为 `🔄 学习中` 的模块
2. 告知用户当前进度
3. 给出该模块的**阅读清单**（网页链接，按顺序），要求用户读完再来
4. 等用户说"读完了"

### 每个模块的阅读清单

**模块 1 - 资源加载方式总览**
1. [Unity 官方 - 最佳实践：Resources 文件夹](https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity2.html)
2. [Unity 官方 - AssetBundle 介绍](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)
3. [Unity 官方 - AssetBundle 工作原理](https://docs.unity3d.com/Manual/AssetBundles-Native.html)
4. [Unity 官方 - Addressables 入门](https://docs.unity3d.com/Packages/com.unity.addressables@latest/manual/index.html)
5. [Unity 官方 - Addressables 内存管理](https://docs.unity3d.com/Packages/com.unity.addressables@latest/manual/MemoryManagement.html)
6. [YooAsset 官方文档 - 快速开始](https://www.yooasset.com/docs/guide-editor/QuickStart)
7. [YooAsset 官方文档 - 资源系统](https://www.yooasset.com/docs/guide-runtime/CodeTutorial1)

*后续模块的阅读清单在模块完成时补充*

### 用户说"读完了"
1. 出题：10-30 道题，由浅入深
2. 每道题等用户回答后再给出下一道，**不要一次性出所有题**
3. 用户答完一道后：
   - 答对：简短确认，继续下一题
   - 答错：给出正确答案 + 底层原理说明，记录到 `wrong_answers.md`
4. 所有题出完后，给出本模块得分总结

### 模块完成后
1. 更新 `progress.md`：
   - 当前模块状态改为 `✅ 完成`，填写得分
   - 下一个模块状态改为 `🔄 学习中`
2. 为下一个模块生成学习资料文件 `module{N}_*.md`
3. git commit + push 同步到 GitHub
4. 告知用户下一个模块是什么

---

## 出题规范

- **由浅入深**：概念题 → 原理题 → 对比题 → 场景分析题
- **每道题单独发**，等待用户回答
- 题型多样：选择、判断、简答、代码分析
- 重点考察**原理和底层机制**，不只是用法记忆

## 错题记录格式

```markdown
### 题目：{题目内容}
**你的答案**：{用户答案}
**正确答案**：{正确答案}
**原理说明**：{为什么，底层是怎么运作的}
**日期**：{YYYY-MM-DD}
```

## git 同步规范

每次以下情况触发 commit + push：
- 模块完成（progress.md 更新）
- 有新错题（wrong_answers.md 更新）
- 新模块资料生成

commit message 格式：
- `progress: 模块{N}完成，得分{X}/10`
- `wrong: 模块{N}新增{X}道错题`
- `content: 生成模块{N}学习资料`
