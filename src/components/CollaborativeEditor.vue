<!-- 文件路径: src/App.vue -->
<template>
  <div class="editor-container">
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
    <editor-content :editor="editor" />
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

// --- 用户信息和光标颜色 ---
const userColor = '#' + Math.floor(Math.random()*16777215).toString(16).padStart(6, '0');
const userName = 'User ' + Math.floor(Math.random() * 100);

// --- Tiptap 编辑器实例 ---
const editor = useEditor({
  extensions: [
    StarterKit.configure({
      history: false, // 使用 Yjs 的历史记录
    }),
    Collaboration.configure({
      document: ydoc,
    }),
    CollaborationCursor.configure({
      provider: provider,
      user: {
        name: userName,
        color: userColor,
      },
    }),
  ],
});

// 组件卸载时清理资源
onUnmounted(() => {
  if (provider) {
    provider.destroy();
  }
  if (ydoc) {
    ydoc.destroy();
  }
});
</script>

<style>
/*
  为了简化，这里使用全局样式。
  注意：scoped 样式对 Tiptap 渲染的 HTML 不起作用，
  因为它们不是组件模板的一部分。
*/

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  margin: 0;
  background-color: #f0f2f5;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

#app {
  width: 100%;
}

.editor-container {
  max-width: 900px;
  margin: 2rem auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
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
}

.toolbar button {
  padding: 4px 8px;
  border: 1px solid #ddd;
  background-color: #fff;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.toolbar button:hover {
  background-color: #f0f0f0;
}

.toolbar button.is-active {
  background-color: #333;
  color: #fff;
}

.ProseMirror {
  min-height: 400px;
  padding: 1rem;
  background-color: #fff;
  border-radius: 5px;
  border: 1px solid #ccc;
  outline: none;
}

.ProseMirror:focus {
  border-color: #66afe9;
}

/* --- 协同光标样式 --- */
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
