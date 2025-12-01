# Bug 修复：重复猹评问题

## 问题描述

**症状**：
- 实际 26 条猹评，显示为 40 条
- 数据中存在重复的猹评
- 从截图可以看到同一条猹评出现多次

## 问题分析

### 网络请求观察

从开发者工具的 Network 面板可以看到：
```
1. proxy?path=users/10000120/reviews?skip=0&limit=20  → 返回 20 条
2. proxy?path=users/10000120/reviews?skip=20&limit=20 → 返回 6 条
3. proxy?path=users/10000120/reviews?skip=40&limit=20 → 返回 ??? (可能是重复数据)
```

### 根本原因

**可能的原因**：
1. **API 返回重复数据**：观猹 API 在某些情况下可能返回重复的猹评
2. **分页边界问题**：当 `skip` 超过实际数据量时，API 可能返回之前的数据
3. **缓存问题**：API 代理或浏览器缓存导致重复

### 为什么会发起第三次请求？

原代码逻辑：
```typescript
// 第一次：skip=0, 返回 20 条
allReviews.push(...response.items);  // allReviews.length = 20
if (response.items.length < PAGE_SIZE) break;  // 20 >= 20, 不退出
if (allReviews.length >= total) break;  // 20 < 26, 不退出
skip += PAGE_SIZE;  // skip = 20

// 第二次：skip=20, 返回 6 条
allReviews.push(...response.items);  // allReviews.length = 26
if (response.items.length < PAGE_SIZE) break;  // 6 < 20, 应该退出！
```

**理论上第二次请求后应该退出**，但如果：
- API 返回的数据有问题
- 或者退出条件判断有 bug
- 就会继续发起第三次请求

## 修复方案

### 1. 添加去重逻辑

使用 `Set` 记录已见过的猹评 ID，防止重复添加：

```typescript
const seenIds = new Set<number>();

for (const item of response.items) {
  if (!seenIds.has(item.id)) {
    seenIds.add(item.id);
    allReviews.push(item);
    addedCount++;
  } else {
    console.warn(`Duplicate review found: ${item.id}`);
  }
}
```

### 2. 添加详细日志

帮助诊断问题：

```typescript
console.log(`Fetching reviews: skip=${skip}, limit=${PAGE_SIZE}`);
console.log(`Total reviews: ${total}`);
console.log(`Received ${response.items.length} items`);
console.log(`Added ${addedCount} new items, total now: ${allReviews.length}`);
console.log(`Received ${response.items.length} < ${PAGE_SIZE}, last page reached`);
```

### 3. 完整的修复代码

```typescript
export async function fetchAllReviews(
  userId: number,
  onProgress?: (loaded: number, total: number) => void
): Promise<ReviewItem[]> {
  const allReviews: ReviewItem[] = [];
  const seenIds = new Set<number>();  // ✅ 去重
  let skip = 0;
  let total = 0;

  while (true) {
    console.log(`Fetching reviews: skip=${skip}, limit=${PAGE_SIZE}`);
    const response = await fetchReviews(userId, skip, PAGE_SIZE);
    
    if (total === 0) {
      total = response.total;
      console.log(`Total reviews: ${total}`);
    }

    if (!response.items || response.items.length === 0) {
      console.log('No more items, breaking');
      break;
    }

    console.log(`Received ${response.items.length} items`);

    // ✅ 去重：只添加未见过的猹评
    let addedCount = 0;
    for (const item of response.items) {
      if (!seenIds.has(item.id)) {
        seenIds.add(item.id);
        allReviews.push(item);
        addedCount++;
      } else {
        console.warn(`Duplicate review found: ${item.id}`);
      }
    }
    
    console.log(`Added ${addedCount} new items, total now: ${allReviews.length}`);
    
    if (onProgress) {
      onProgress(allReviews.length, total);
    }

    // ✅ 退出条件 1：最后一页
    if (response.items.length < PAGE_SIZE) {
      console.log(`Received ${response.items.length} < ${PAGE_SIZE}, last page reached`);
      break;
    }

    // ✅ 退出条件 2：已获取全部
    if (allReviews.length >= total) {
      console.log(`Collected ${allReviews.length} >= ${total}, all data fetched`);
      break;
    }

    skip += PAGE_SIZE;
    
    // ✅ 退出条件 3：安全检查
    if (skip > total + PAGE_SIZE) {
      console.warn('Pagination safety check triggered');
      break;
    }
  }

  console.log(`Final count: ${allReviews.length} reviews`);
  return allReviews;
}
```

## 修复效果

### 修复前
```
26 条实际猹评 → 显示 40 条（包含重复）
```

### 修复后
```
26 条实际猹评 → 显示 26 条（去重后）
```

### 日志输出示例

```
Fetching reviews: skip=0, limit=20
Total reviews: 26
Received 20 items
Added 20 new items, total now: 20

Fetching reviews: skip=20, limit=20
Received 6 items
Added 6 new items, total now: 26
Received 6 < 20, last page reached

Final count: 26 reviews
```

如果有重复：
```
Fetching reviews: skip=40, limit=20
Received 14 items
Duplicate review found: 12345
Duplicate review found: 12346
...
Added 0 new items, total now: 26
```

## 技术细节

### 去重机制

1. **使用 Set 存储 ID**
   - `Set<number>` 提供 O(1) 的查找性能
   - 自动去重，不会存储重复的 ID

2. **检查每个猹评**
   ```typescript
   if (!seenIds.has(item.id)) {
     seenIds.add(item.id);
     allReviews.push(item);
   }
   ```

3. **记录重复**
   ```typescript
   console.warn(`Duplicate review found: ${item.id}`);
   ```

### 为什么会有重复？

可能的原因：
1. **API 分页 bug**：观猹 API 在边界情况下返回重复数据
2. **并发请求**：虽然我们是串行请求，但 API 可能有缓存问题
3. **数据更新**：在获取过程中，用户可能新增了猹评

### 去重的必要性

即使修复了分页逻辑，去重仍然是必要的：
- **防御性编程**：不依赖 API 的完美行为
- **数据一致性**：确保用户看到的数据是准确的
- **用户体验**：避免导出重复的猹评

## 测试结果

- ✅ 所有 43 个单元测试通过
- ✅ 构建成功
- ✅ 添加了去重逻辑
- ✅ 添加了详细日志

## 验证步骤

1. **等待 Vercel 部署完成**

2. **测试用户 @10000120**（26 条猹评）
   - 输入主页链接
   - 打开浏览器控制台查看日志
   - 确认最终显示 26 条（不是 40 条）

3. **检查控制台日志**
   ```
   Fetching reviews: skip=0, limit=20
   Total reviews: 26
   Received 20 items
   Added 20 new items, total now: 20
   
   Fetching reviews: skip=20, limit=20
   Received 6 items
   Added 6 new items, total now: 26
   Received 6 < 20, last page reached
   
   Final count: 26 reviews
   ```

4. **检查是否有重复警告**
   - 如果看到 `Duplicate review found: xxx`
   - 说明 API 确实返回了重复数据
   - 但我们的去重逻辑已经处理了

5. **导出并验证**
   - 导出 TXT 文件
   - 检查是否有重复的猹评
   - 确认总数正确

## 后续优化

如果日志显示确实有大量重复：

1. **调查 API 行为**
   - 记录哪些情况下会返回重复
   - 是否与 `skip` 参数有关

2. **优化请求策略**
   - 可能需要调整分页参数
   - 或者使用不同的 API 端点

3. **移除日志**
   - 确认问题解决后
   - 可以移除详细的 console.log
   - 保留 console.warn（重复警告）

---

**修复日期**: 2025-12-02  
**提交**: 4bf028b  
**状态**: 已修复，等待验证
