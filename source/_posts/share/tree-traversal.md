---
title: 算法实战系列：树形结构遍历
date: 2023-03-20 14:04:49
updated: 2023-03-20 14:04:49
tags:
  - 深度优先遍历
  - 广度优先遍历
categories:
  - 技术分享
  - 算法
---

### Before...

- 前几天在一家独角兽公司复试，本以为会很顺利，但随着最后技术面的一道算法题失利而告终。（面试官：最后再考你一道题吧，不会很难的... 遍历一颗树，找到符合某个要求的节点。我： emmm 确实感觉挺简单的，平时也会遇到这类型的需求，但不知道是脑子瓦特了还是真的因为紧张，一时间没有想到最优实现，最后通过深度遍历完成，但并不符合面试官预期...他想要的是广度优先遍历实现）

- 只能说是准备不充分，稳稳拿下的面试因为这两年工作而遗忘的东西而失败。今天全当是复习了...、

- **持续更新中**..

<!-- more -->

### 广度优先遍历

> `使用广度优先遍历查找某一个节点，需要使用队列来辅助实现。`

具体实现：

- 创建一个队列，将根节点入队。
- 当队列不为空时，取出队列中的队首元素。
- 如果当前节点的值等于目标值，返回当前节点。
- 否则将当前节点的所有子节点加入队列。
- 重复步骤 2-4，直到队列为空。

```typescript
interface TreeNode {
  value: number;
  children: TreeNode[];
}

function bfsFindNode(root: TreeNode, targetValue: number): TreeNode | null {
  const queue: TreeNode[] = [root]; // 创建一个队列，将根节点入队

  while (queue.length > 0) {
    const currentNode = queue.shift()!; // 取出队列中的队首元素

    if (currentNode.value === targetValue) {
      return currentNode; // 如果当前节点的值等于目标值，返回当前节点
    }

    queue.push(...currentNode.children); // 将当前节点的所有子节点加入队列
  }

  return null; // 队列为空，未找到目标节点，返回null
}
```

创建一个队列，将根节点入队，然后使用一个`while`循环来遍历队列中的元素。每次取出队首元素，并判断其值是否等于目标值，如果相等，则返回该节点；否则将该节点的所有子节点加入队列。当队列为空时，表示未找到目标节点，返回`null`。

### 深度优先遍历

> `使用递归实现深度优先遍历。`

- 访问当前节点，并判断当前节点的值是否等于目标值。
- 如果当前节点的值等于目标值，返回当前节点。
- 否则对当前节点的每个子节点，递归调用深度优先遍历函数。
- 重复步骤 1-3，直到遍历完整棵树。

```typescript
interface TreeNode {
  value: number;
  children: TreeNode[];
}

function dfsFindNode(root: TreeNode, targetValue: number): TreeNode | null {
  if (root.value === targetValue) {
    return root; // 当前节点的值等于目标值，返回当前节点
  }

  for (const node of root.children) {
    const result = dfsFindNode(node, targetValue); // 递归查找子节点

    if (result !== null) {
      return result; // 找到目标节点，直接返回
    }
  }

  return null; // 未找到目标节点，返回 null
}
```

首先访问当前节点，并判断其值是否等于目标值。如果等于目标值，直接返回当前节点；否则对当前节点的每个子节点，递归调用深度优先遍历函数，并返回找到的目标节点。当遍历完整棵树后，仍未找到目标节点，则返回`null`。

```typescript
const root: TreeNode = {
  value: 1,
  children: [
    {
      value: 2,
      children: [
        { value: 4, children: [] },
        { value: 5, children: [] },
      ],
    },
    {
      value: 3,
      children: [
        { value: 6, children: [] },
        { value: 7, children: [] },
      ],
    },
  ],
};

const targetNode = dfsFindNode(root, 6);
console.log(targetNode); // { value: 6, children: [] }
```
