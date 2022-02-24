---
title: "Storybook Vite Vue3 Tsx"
description: 
date: 2022-02-24T19:08:51+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

## 前言
需要有一个可实时浏览Vue3 Tsx组件库的功能，发现Storybook是最符合需求的工具。
本文章介绍了如何使用Storybook和Vite从零搭建一个Vue3 Tsx组件库。
## 快速开始

```bash
# 使用vite快速创建vue-ts项目
yarn create vite storybook-vite-vue-tsx --template vue-ts
cd storybook-vite-vue-tsx
# 安装依赖
yarn install
# 添加storybook vite builder plugin
npx sb@next init --builder storybook-builder-vite
# 删除自带的stories
rm -rf src/stories
# 启动
yarn storybook
# 构建
yarn build-storybook
```

## 添加JSX支持

```bash
# 添加plugin-vue-jsx
yarn add @vitejs/plugin-vue-jsx
# 配置内容参考配置段落
```

## 配置
storybook的主要配置内容在.storybook/main.js

    vite的配置需要在.storybook/main.js的viteFinal配置项进行设置，vite.config.ts实际不起作用

```ts
const vue = require("@vitejs/plugin-vue").default;
// jsx插件
const vueJsx = require("@vitejs/plugin-vue-jsx").default;
const path = require("path");

module.exports = {
  stories: ["../src/**/*.stories.mdx", "../src/**/*.stories.@(js|jsx|ts|tsx)"],
  addons: ["@storybook/addon-links", "@storybook/addon-essentials"],
  framework: "@storybook/vue3",
  features: {
    interactionsDebugger: true,
  },
  core: {
    builder: "storybook-builder-vite",
  },
  async viteFinal(config, { configType }) {
    config.plugins.push(vue());
    config.plugins.push(vueJsx());
    config.resolve = {
      alias: {
        vue: "vue/dist/vue.esm-bundler.js",
        "@": path.resolve(__dirname, "../src"), // 设置别名
      },
    };
    config.build = {
      assetsDir: "assets",
    };   
    return config;
  },
};
```

## 官方Task组件示例

创建 `src/components/Task/index.tsx` 文件
```tsx
import { defineComponent, onMounted, Ref, ref, watch, computed, PropType } from "vue";
export interface TaskPropType {
  id: string;
  state: string;
  title: string;
  updatedAt?: Date;
}
export default defineComponent({
  name: "Task",
  props: {
    task: {
      type: Object as PropType<TaskPropType>,
      required: true,
      default: () => ({ id: "", state: "", title: "" }),
      validator: (task: TaskPropType) => ["id", "state", "title"].every((key) => key in task),
    },
  },
  emits: ["archive-task", "pin-task"],
  setup(props, { emit, slots }) {
    const classes = computed(() => ({
      "list-item TASK_INBOX": props.task.state === "TASK_INBOX",
      "list-item TASK_PINNED": props.task.state === "TASK_PINNED",
      "list-item TASK_ARCHIVED": props.task.state === "TASK_ARCHIVED",
    }));
    const isChecked = computed(() => props.task.state === "TASK_ARCHIVED");
    return () => {
      return (
        <div class={classes.value}>
          <label class="checkbox">
            <input type="checkbox" checked={isChecked.value} disabled name="checked" />
            <span
              class="checkbox-custom"
              onClick={() => {
                emit("archive-task", props.task.id);
              }}
              aria-label={"archiveTask-" + props.task.id}
            ></span>
          </label>
          <div class="ttile">
            <input type="text" value={props.task.title} readonly placeholder="Input title" />
          </div>
          <div class="actions">
            {!isChecked.value && (
              <a
                onClick={() => {
                  emit("pin-task", props.task.id);
                }}
              >
                <span class="icon-star"></span>
              </a>
            )}
          </div>
        </div>
      );
    };
  },
});
```
创建`src/components/index.stories.tsx`文件

```tsx
import Task, { TaskPropType } from ".";
import { action } from "@storybook/addon-actions";
import { Story } from "@storybook/vue3";

export default {
  component: Task,
  title: "Task",
  //👇 Our exports that end in "Data" are not stories.
  excludeStories: /.*Data$/,
  //👇 Our events will be mapped in Storybook UI
  argTypes: {
    onPinTask: { action: true },
    onArchiveTask: { action: true },
  },
};
export const actionsData = {
  onPinTask: action("pin-task"),
  onArchiveTask: action("archive-task"),
};

interface Args {
  task: TaskPropType;
  onArchiveTask: (taskId: string) => void;
  onPinTask: (taskId: string) => void;
}
// export const Basic = () => <Task task={{ id: "1", state: "Basic", title: "Basic", updateAt: new Date() }}></Task>;

// const Template: Story<TaskArgs> = (props) => ({
//   setup() {
//     return { ...actionsData };
//   },
//   render() {
//     return <Task {...props} />;
//   },
// });
const Template: Story<Args> = (props) => (
  <Task task={props.task} onArchive-task={props.onArchiveTask} onPin-task={props.onPinTask}></Task>
);

export const Default = Template.bind({});
Default.args = {
  task: {
    id: "1",
    title: "Test Task",
    state: "TASK_INBOX",
    updatedAt: new Date(2018, 0, 1, 9, 0),
  },
  onArchiveTask: actionsData.onArchiveTask,
  onPinTask: actionsData.onPinTask,
};

export const Pinned = Template.bind({});
Pinned.args = {
  task: {
    ...Default.args.task!,
    state: "TASK_PINNED",
  },
};

export const Archived = Template.bind({});
Archived.args = {
  task: {
    ...Default.args.task!,
    state: "TASK_ARCHIVED",
  },
};
```

启动 `yarn storybook`

## 源码

https://github.com/fdxxw/storybook-vite-vue-tsx

## 参考

1. https://storybook.js.org/tutorials/intro-to-storybook/vue/zh-CN/get-started/