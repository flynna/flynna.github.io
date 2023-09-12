---
title: 基于 Antd-Table 二次封装的 ResizeTable 组件
date: 2023-04-12 18:47:25
updated: 2023-04-12 18:47:25
tags:
  - ResizeTable
  - react
categories:
  - 技术分享
  - 组件封装
---

`antd` 提供的 `table` 组件本身是不支持改变列宽的，在某些场景下，用户希望通过自己调节查看列详情。

类似`vue-element`又支持了这个功能，所以想着封装一版`react`可拖拽列宽的表格组件。

<!-- more -->

### ResizeTable 组件实现

`组件接收的 Props 同 Table 一致`.

```tsx
import { delayMS } from '@/utils/utils';
import { Table } from 'antd';
import type { ColumnType, TableProps } from 'antd/lib/table';
import type { TableRowSelection } from 'antd/lib/table/interface';
import React, { useEffect, useMemo, useState } from 'react';
import { Resizable } from 'react-resizable';

interface ResizeTableProps extends TableProps<any> {
  columns: ColumnType<any>[]; // any 取决于 row 数据类型
}

const ResizableTitle = (props: { [x: string]: any; onResize: any; width: any }) => {
  const { onResize, width, onClick, ...restProps } = props;

  // fix: 修复拖拽改变列宽操作与排序器冲突
  const [resizing, setResizing] = useState(false);

  if (!width) {
    return <th {...restProps} />;
  }

  return (
    <Resizable
      width={width}
      height={0}
      onResizeStart={() => setResizing(true)}
      onResizeStop={async () => {
        await delayMS(0); // setTimeout 0
        setResizing(false);
      }}
      onResize={onResize}
      // draggableOpts={{ enableUserSelectHack: false }}
    >
      <th
        onClick={(...args) => {
          if (!resizing && onClick) {
            onClick(...args);
          }
        }}
        {...restProps}
      />
    </Resizable>
  );
};

const ResizeTable: React.FC<ResizeTableProps> = (props) => {
  const [columns, setColumns] = useState<ColumnType<API.RowDataInfo>[]>([]);

  useEffect(() => {
    setColumns(props.columns || []);
  }, [props.columns]);

  const handleResize =
    (index: number) =>
    (e: Event, { size }: any) => {
      e.stopImmediatePropagation();
      // e.preventDefault();

      setColumns((cols) => {
        const nextColumns = [...cols];
        nextColumns[index] = {
          ...nextColumns[index],
          width: size.width,
        };
        return nextColumns;
      });
    };

  const customColumns = useMemo(
    () =>
      columns.map((col, index) => ({
        ...col,
        onHeaderCell: (column: { width: any }) => ({
          width: column.width,
          onResize: handleResize(index),
        }),
      })),
    [columns],
  );

  return (
    <Table
      {...props}
      columns={customColumns as any}
      components={{ header: { cell: ResizableTitle } }}
    />
  );
};

export default React.memo(ResizeTable);
```
