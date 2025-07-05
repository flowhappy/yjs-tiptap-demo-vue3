<template>
  <div class="editor-wrapper">
    <!-- Tiptap 工具栏 -->
    <div v-if="editor" class="toolbar">
      <button @click="editor.chain().focus().toggleBold().run()" :class="{ 'is-active': editor.isActive('bold') }">
        Bold
      </button>
      <button @click="editor.chain().focus().toggleItalic().run()" :class="{ 'is-active': editor.isActive('italic') }">
        Italic
      </button>
      <button @click="editor.chain().focus().toggleHeading({ level: 1 }).run()" :class="{ 'is-active': editor.isActive('heading', { level: 1 }) }">
        H1
      </button>
      <button @click="editor.chain().focus().toggleHeading({ level: 2 }).run()" :class="{ 'is-active': editor.isActive('heading', { level: 2 }) }">
        H2
      </button>
      <button @click="editor.chain().focus().toggleCodeBlock().run()" :class="{ 'is-active': editor.isActive('codeBlock') }">
        Code
      </button>
    </div>

    <!-- Tiptap 编辑器实例 -->
    <editor-content :editor="editor" class="editor-content-area" />
  </div>
</template>

<script setup>
import { onUnmounted } from 'vue';

// Tiptap 和 Yjs 相关导入
import { useEditor, EditorContent } from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';
import Collaboration from '@tiptap/extension-collaboration';
import CollaborationCursor from '@tiptap/extension-collaboration-cursor';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

// --- Yjs 设置 ---
const ydoc = new Y.Doc();
const roomName = 'tiptap-demo-room';
const websocketUrl = 'ws://localhost:1234';
const provider = new WebsocketProvider(websocketUrl, roomName, ydoc);

// --- 用户信息 ---
const userColor = '#' + Math.floor(Math.random()*16777215).toString(16).padStart(6, '0');
const userName = 'User ' + Math.floor(Math.random() * 100);

// --- Tiptap 编辑器实例 ---
const editor = useEditor({
  extensions: [
    StarterKit.configure({ history: false }),
    Collaboration.configure({ document: ydoc }),
    CollaborationCursor.configure({
      provider: provider,
      user: { name: userName, color: userColor },
    }),
  ],
});

onUnmounted(() => {
  if (provider) provider.destroy();
  if (ydoc) ydoc.destroy();
});
</script>

<style>
/*
  重要提示：这里的样式没有使用 `scoped`。
  因为 Tiptap 渲染的 .ProseMirror 元素不在 Vue 组件的模板内，
  `scoped` 样式无法应用到它。所以我们将这些样式保持全局，但通过父选择器 .editor-wrapper 限定作用域。
*/
.editor-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh; /* 撑满父容器高度 */
  background-color: #f9f9f9;
  padding: 2rem;
  box-sizing: border-box;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 16px;
  padding: 8px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 5px;
  flex-shrink: 0; /* 防止工具栏被压缩 */
}

.toolbar button {
  padding: 4px 8px;
  border: 1px solid #ddd;
  background-color: #fff;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.toolbar button.is-active {
  background-color: #333;
  color: #fff;
}

.editor-content-area {
  flex-grow: 1; /* 占据所有剩余空间 */
  overflow-y: auto; /* 内容过多时滚动 */
  background-color: #fff;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.ProseMirror {
  padding: 1rem;
  outline: none;
  height: 100%; /* 继承父容器高度 */
}

.ProseMirror:focus {
  border-color: transparent;
  box-shadow: none;
}

/* 协同光标样式 */
.collaboration-cursor__caret {
  position: relative;
  margin-left: -1px;
  margin-right: -1px;
  border-left: 1px solid #0D0D0D;
  border-right: 1px solid #0D0D0D;
  word-break: normal;
  pointer-events: none;
}

.collaboration-cursor__label {
  position: absolute;
  top: -1.4em;
  left: -1px;
  font-size: 12px;
  font-style: normal;
  font-weight: 600;
  line-height: normal;
  user-select: none;
  pointer-events: none;
  padding: 0.1rem 0.3rem;
  border-radius: 3px 3px 3px 0;
  white-space: nowrap;
}
</style>
