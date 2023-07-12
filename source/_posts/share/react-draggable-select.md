---
title: 可对已选项拖拽排序的 React 下拉选择组件
date: 2023-06-20 19:58:47
updated: 2023-06-20 19:58:47
tags:
  - Select
  - draggable
  - react
categories:
  - 技术分享
  - 功能实现
---

基于最近项目上的一个需求：对已经选择的多个选项进行排序。由于`select`组件本身没有支持，所以单独封装实现。

> 基于 `react-beautiful-dnd` 模块实现的一个拖拽实例

<!-- more -->

### 源码实现

#### `DraggableSelect` 组件

`接受的 Props 同 Select 组件.`

```tsx
import type { SelectProps } from 'antd';
import { Select } from 'antd';
import type { LabeledValue, SelectValue } from 'antd/es/select';
import type { CSSProperties } from 'react';
import React from 'react';
import type { DraggableProvidedDraggableProps, DropResult } from 'react-beautiful-dnd';
import { DragDropContext, Draggable, Droppable } from 'react-beautiful-dnd';
import TagItem from './components/TagItem';

const getItemStyle = (
  isDragging: boolean,
  draggableStyle: DraggableProvidedDraggableProps['style'],
): CSSProperties => ({
  userSelect: 'none',
  opacity: isDragging ? 0.5 : 1,
  ...draggableStyle,
});

const reorder = (list: SelectValue, startIndex: number, endIndex: number) => {
  const result = Array.from(list as any[]);
  const [removed] = result.splice(startIndex, 1);
  result.splice(endIndex, 0, removed);

  return result;
};

const DraggableSelect = ({
  onChange,
  value,
  tagRender,
  ...props
}: SelectProps<any> & { onChange?: (value: SelectValue) => void }) => {
  const onDragEnd = (result: DropResult) => {
    const {
      destination,
      source: { index: dragIndex },
    } = result;

    if (destination) {
      const items = reorder(value, dragIndex, destination.index);

      onChange?.(items);
    }
  };

  return (
    <DragDropContext onDragEnd={onDragEnd}>
      <Droppable droppableId='list' direction='horizontal'>
        {(provided) => (
          <div {...provided.droppableProps} ref={provided.innerRef}>
            <Select
              style={{ width: '100%' }}
              tagRender={(tagProps) => (
                <Draggable
                  key={tagProps.value as string}
                  draggableId={tagProps.value as string}
                  index={value.findIndex((item: SelectValue) =>
                    typeof item === 'string'
                      ? item === tagProps.value
                      : (item as LabeledValue).value === tagProps.value,
                  )}>
                  {(pd, snapshot) => (
                    <div
                      ref={pd.innerRef}
                      {...pd.draggableProps}
                      {...pd.dragHandleProps}
                      style={getItemStyle(snapshot.isDragging, pd.draggableProps.style)}
                      onMouseDown={(e) => {
                        e.stopPropagation();
                      }}>
                      {tagRender ? tagRender(tagProps) : <TagItem {...tagProps} />}
                    </div>
                  )}
                </Draggable>
              )}
              onChange={onChange}
              value={value}
              {...props}
            />
            {/* FIX: 添加 provided.placeholder 解决 warning 警告，display：none 不使用占位符 */}
            <div style={{ display: 'none' }}>{provided.placeholder}</div>
          </div>
        )}
      </Droppable>
    </DragDropContext>
  );
};

export default DraggableSelect;
```

#### `TagItem` 组件

```tsx
import { CloseOutlined } from '@ant-design/icons';
import type { CustomTagProps } from 'rc-select/lib/interface/generator';
import type { CSSProperties } from 'react';
import React from 'react';

export type TagItemProps = {
  label?: React.ReactNode;
  originLabel?: string;
  style?: CSSProperties;
  dataHandlerId?: string;
  onMouseDown?: React.MouseEventHandler;
} & CustomTagProps;

const TagItem: React.ForwardRefRenderFunction<any, TagItemProps> = (
  { label, originLabel, dataHandlerId, onMouseDown, onClose, style },
  ref,
) => (
  <span
    className='ant-select-selection-item'
    title={originLabel as string}
    style={style}
    ref={ref}
    onMouseDown={onMouseDown}
    data-handler-id={dataHandlerId}>
    <span className='ant-select-selection-item-content'>{label}</span>
    <span className='ant-select-selection-item-remove' unselectable='on' aria-hidden='true'>
      <CloseOutlined onClick={onClose} />
    </span>
  </span>
);

export default React.forwardRef(TagItem);
```
