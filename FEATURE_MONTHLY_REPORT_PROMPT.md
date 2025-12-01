# 新功能：月报配方按钮

## 功能概述

在页面头部添加了"📝 月报配方"按钮，点击后弹出包含完整提示词的弹窗，用户可以一键复制到剪贴板，方便将导出的数据配合提示词使用AI生成月报。

## 功能特性

### 1. 月报配方按钮

- **位置**：页面头部，标题下方
- **样式**：次要按钮样式（白底绿边）
- **图标**：📝 表示文档/配方

### 2. 弹窗展示

**弹窗包含**：
- **标题**：📝 观猹AI洞察月报配方
- **内容区**：完整的提示词文本，使用等宽字体显示
- **关闭按钮**：右上角 × 按钮
- **操作按钮**：📋 复制到剪贴板

**交互方式**：
- 点击"月报配方"按钮打开
- 点击 × 按钮关闭
- 点击弹窗外部关闭
- 按 ESC 键关闭

### 3. 一键复制

- 点击"📋 复制到剪贴板"按钮
- 自动复制完整提示词到剪贴板
- 显示"✓ 已复制"提示（2秒后自动消失）
- 使用现代 Clipboard API

## 提示词内容

提示词来自"观猹AI洞察月报"配方，包含以下部分：

### 核心结构

1. **角色与背景**
   - 定位为"首席认知官"与"智力镜像"
   - 处理观猹平台的评论和讨论记录

2. **核心任务**
   - 清洗、重组与深度分析
   - 生成《观猹AI洞察月报》

3. **输出框架**
   - 📊 能量热力图
   - 🧠 智力焦点与成果
   - ✨ 关键洞见
   - 🕵️ 盲区、矛盾与深层阻碍
   - 🚀 下月导航系统

4. **思考协议**
   - 降噪、聚类、归因、反直觉检查

5. **沟通规则**
   - 拒绝平庸、镜像原则、审慎归纳

## 使用场景

### 典型工作流

1. **导出数据**
   - 使用工具导出某个月的猹评或讨论
   - 保存为 TXT 文件

2. **获取提示词**
   - 点击"月报配方"按钮
   - 点击"复制到剪贴板"

3. **生成月报**
   - 打开 AI 工具（如 ChatGPT、Claude、Gemini）
   - 粘贴提示词
   - 附上导出的数据
   - 让 AI 生成月报

### 示例对话

```
用户：[粘贴提示词]

[粘贴导出的猹评数据]

AI：根据您提供的数据，我将生成《观猹AI洞察月报》...

📊 能量热力图
...
```

## 技术实现

### 1. HTML 结构

```html
<!-- 按钮 -->
<button id="prompt-btn" class="secondary-btn">📝 月报配方</button>

<!-- 弹窗 -->
<div id="prompt-modal" class="modal hidden">
  <div class="modal-content">
    <div class="modal-header">
      <h2>📝 观猹AI洞察月报配方</h2>
      <button id="close-modal" class="close-btn">&times;</button>
    </div>
    <div class="modal-body">
      <pre id="prompt-text" class="prompt-text"></pre>
    </div>
    <div class="modal-footer">
      <button id="copy-prompt-btn" class="primary-btn">📋 复制到剪贴板</button>
      <span id="copy-success" class="copy-success hidden">✓ 已复制</span>
    </div>
  </div>
</div>
```

### 2. CSS 样式

**次要按钮**：
```css
.secondary-btn {
  background: var(--card-bg);
  color: var(--primary);
  border: 2px solid var(--primary);
}

.secondary-btn:hover {
  background: var(--primary);
  color: white;
}
```

**弹窗样式**：
```css
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: var(--card-bg);
  border-radius: 12px;
  max-width: 800px;
  width: 100%;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
}
```

**提示词文本**：
```css
.prompt-text {
  background: var(--bg);
  padding: 1.5rem;
  border-radius: 8px;
  font-family: 'Courier New', monospace;
  font-size: 0.875rem;
  line-height: 1.6;
  white-space: pre-wrap;
  word-wrap: break-word;
}
```

### 3. JavaScript 逻辑

**提示词常量**：
```typescript
const PROMPT_TEMPLATE = `观猹AI洞察月报
...`;
```

**显示弹窗**：
```typescript
function showPromptModal() {
  promptText.textContent = PROMPT_TEMPLATE;
  promptModal.classList.remove('hidden');
  document.body.style.overflow = 'hidden';
}
```

**复制到剪贴板**：
```typescript
async function copyPromptToClipboard() {
  try {
    await navigator.clipboard.writeText(PROMPT_TEMPLATE);
    copySuccess.classList.remove('hidden');
    setTimeout(() => {
      copySuccess.classList.add('hidden');
    }, 2000);
  } catch (error) {
    showError('复制失败，请手动选择文本复制');
  }
}
```

**事件绑定**：
```typescript
promptBtn.addEventListener('click', showPromptModal);
closeModal.addEventListener('click', hidePromptModal);
copyPromptBtn.addEventListener('click', copyPromptToClipboard);

// 点击外部关闭
promptModal.addEventListener('click', (e) => {
  if (e.target === promptModal) {
    hidePromptModal();
  }
});

// ESC键关闭
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    hidePromptModal();
  }
});
```

## 用户体验优化

### 1. 视觉反馈

- **按钮悬停**：颜色反转（白底绿字 → 绿底白字）
- **复制成功**：显示"✓ 已复制"提示
- **弹窗动画**：平滑的淡入淡出效果

### 2. 交互便利性

- **多种关闭方式**：
  - 点击 × 按钮
  - 点击弹窗外部
  - 按 ESC 键

- **防止页面滚动**：
  - 弹窗打开时禁用背景滚动
  - 关闭时恢复滚动

### 3. 响应式设计

- **桌面端**：弹窗最大宽度 800px
- **移动端**：弹窗占满屏幕宽度，高度 95vh
- **字体大小**：移动端自动缩小以适应屏幕

### 4. 可访问性

- **语义化 HTML**：使用 `<pre>` 标签保留格式
- **键盘导航**：支持 ESC 键关闭
- **清晰的按钮文字**：明确的操作提示

## 浏览器兼容性

### Clipboard API

- ✅ Chrome 63+
- ✅ Firefox 53+
- ✅ Safari 13.1+
- ✅ Edge 79+

**降级处理**：
- 如果复制失败，显示错误提示
- 用户可以手动选择文本复制

## 未来优化

### 1. 提示词管理

- 支持多个提示词模板
- 用户可以自定义提示词
- 保存到本地存储

### 2. 智能填充

- 自动将导出的数据附加到提示词后
- 一键发送到 AI 工具

### 3. 模板变量

- 支持 `{month}`, `{year}` 等变量
- 自动替换为当前筛选的月份

### 4. 分享功能

- 生成提示词分享链接
- 导出为 Markdown 文件

## 测试建议

### 功能测试

- ✅ 点击按钮打开弹窗
- ✅ 弹窗显示完整提示词
- ✅ 点击复制按钮成功复制
- ✅ 显示"已复制"提示
- ✅ 点击 × 关闭弹窗
- ✅ 点击外部关闭弹窗
- ✅ 按 ESC 关闭弹窗

### 兼容性测试

- 测试不同浏览器的复制功能
- 测试移动端的显示效果
- 测试长文本的滚动

### 用户体验测试

- 弹窗是否容易关闭
- 提示词是否清晰可读
- 复制是否方便快捷

---

**功能添加日期**: 2025-12-02  
**提交**: f96d193  
**状态**: 已完成并部署
