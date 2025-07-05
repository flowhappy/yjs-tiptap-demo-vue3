<template>
  <div class="editor-wrapper">
    <!-- 全新升级的工具栏 -->
    <div v-if="editor" class="toolbar">
      <!-- 预览按钮和分组分隔线 -->
      <div class="toolbar-group">
        <button @click="togglePreview" :class="{ 'is-active': isPreviewMode }" title="预览/编辑">
          <component :is="isPreviewMode ? BookOpen : Code" />
        </button>
      </div>

      <!-- 标题下拉菜单 -->
      <div class="toolbar-group">
        <select v-model="activeHeadingLevel" @change="handleHeadingChange" class="heading-select" title="标题">
          <option value="0">段落</option>
          <option value="1">标题 1</option>
          <option value="2">标题 2</option>
          <option value="3">标题 3</option>
        </select>
      </div>

      <!-- 基础文本格式 -->
      <div class="toolbar-group">
        <button @click="editor.chain().focus().toggleBold().run()" :disabled="!editor.can().chain().focus().toggleBold().run()" :class="{ 'is-active': editor.isActive('bold') }" title="加粗">
          <Bold />
        </button>
        <button @click="editor.chain().focus().toggleItalic().run()" :disabled="!editor.can().chain().focus().toggleItalic().run()" :class="{ 'is-active': editor.isActive('italic') }" title="斜体">
          <Italic />
        </button>
        <button @click="editor.chain().focus().toggleStrike().run()" :disabled="!editor.can().chain().focus().toggleStrike().run()" :class="{ 'is-active': editor.isActive('strike') }" title="删除线">
          <Strikethrough />
        </button>
        <button @click="editor.chain().focus().toggleCode().run()" :disabled="!editor.can().chain().focus().toggleCode().run()" :class="{ 'is-active': editor.isActive('code') }" title="行内代码">
          <Code />
        </button>
      </div>

      <!-- 对齐方式 -->
      <div class="toolbar-group">
        <button @click="editor.chain().focus().setTextAlign('left').run()" :class="{ 'is-active': editor.isActive({ textAlign: 'left' }) }" title="左对齐">
          <AlignLeft />
        </button>
        <button @click="editor.chain().focus().setTextAlign('center').run()" :class="{ 'is-active': editor.isActive({ textAlign: 'center' }) }" title="居中对齐">
          <AlignCenter />
        </button>
        <button @click="editor.chain().focus().setTextAlign('right').run()" :class="{ 'is-active': editor.isActive({ textAlign: 'right' }) }" title="右对齐">
          <AlignRight />
        </button>
      </div>

      <!-- 列表和块级元素 -->
      <div class="toolbar-group">
        <button @click="editor.chain().focus().toggleBulletList().run()" :class="{ 'is-active': editor.isActive('bulletList') }" title="项目符号">
          <List />
        </button>
        <button @click="editor.chain().focus().toggleOrderedList().run()" :class="{ 'is-active': editor.isActive('orderedList') }" title="数字列表">
          <ListOrdered />
        </button>
        <button @click="editor.chain().focus().toggleBlockquote().run()" :class="{ 'is-active': editor.isActive('blockquote') }" title="引用">
          <Quote />
        </button>
        <button @click="editor.chain().focus().toggleCodeBlock().run()" :class="{ 'is-active': editor.isActive('codeBlock') }" title="代码块">
          <FileCode />
        </button>
      </div>

      <!-- 插入元素 -->
      <div class="toolbar-group">
        <button @click="editor.chain().focus().setHorizontalRule().run()" title="水平分割线">
          <Minus />
        </button>
      </div>

    </div>

    <!-- 编辑区和预览区 -->
    <div class="content-container">
      <editor-content v-show="!isPreviewMode" :editor="editor" class="editor-content-area" />
      <div v-if="isPreviewMode" v-html="previewContent" class="preview-area ProseMirror"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted, computed, watch } from 'vue';

// Tiptap 和 Yjs
import { useEditor, EditorContent } from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';
import Collaboration from '@tiptap/extension-collaboration';
import CollaborationCursor from '@tiptap/extension-collaboration-cursor';
import TextAlign from '@tiptap/extension-text-align'; // 引入文本对齐扩展
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

// 引入图标
import {
  Bold, Italic, Strikethrough, Code, List, ListOrdered, Quote, Minus,
  AlignLeft, AlignCenter, AlignRight, FileCode, BookOpen
} from 'lucide-vue-next';


// --- 状态管理 ---
const isPreviewMode = ref(false);

// --- Yjs 设置 (保持不变) ---
const ydoc = new Y.Doc();
const roomName = 'tiptap-demo-room';
const websocketUrl = 'ws://localhost:1234';
const provider = new WebsocketProvider(websocketUrl, roomName, ydoc);

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
    // 配置文本对齐扩展
    TextAlign.configure({
      types: ['heading', 'paragraph'],
    }),
  ],
});

// --- 预览逻辑 ---
const previewContent = computed(() => editor.value ? editor.value.getHTML() : '');
const togglePreview = () => {
  isPreviewMode.value = !isPreviewMode.value;
};

// --- 标题下拉菜单逻辑 ---
const activeHeadingLevel = ref(0);

const handleHeadingChange = (event) => {
  const level = parseInt(event.target.value, 10);
  if (level === 0) {
    editor.value.chain().focus().setParagraph().run();
  } else {
    editor.value.chain().focus().toggleHeading({ level: level }).run();
  }
};

// 监听 Tiptap 编辑器状态，更新下拉菜单的显示
watch(() => editor.value?.state.selection, () => {
  if (!editor.value) return;
  for (let i = 1; i <= 3; i++) {
    if (editor.value.isActive('heading', { level: i })) {
      activeHeadingLevel.value = i;
      return;
    }
  }
  activeHeadingLevel.value = 0;
}, { immediate: true, deep: true });


onUnmounted(() => {
  if (provider) provider.destroy();
  if (ydoc) ydoc.destroy();
});
</script>

<style>
/* ... 此前的 .editor-wrapper, .content-container 等样式可以保留 ... */
.editor-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f9f9f9;
  padding: 2rem;
  box-sizing: border-box;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
  padding: 8px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 5px;
  flex-shrink: 0;
}

.toolbar-group {
  display: flex;
  align-items: center;
  padding-right: 8px;
  margin-right: 8px;
  border-right: 1px solid #e0e0e0;
}
.toolbar-group:last-child {
  border-right: none;
  margin-right: 0;
  padding-right: 0;
}

.toolbar button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  padding: 4px;
  border: none;
  background-color: transparent;
  border-radius: 4px;
  cursor: pointer;
  color: #333;
  transition: background-color 0.2s;
}

.toolbar button:hover {
  background-color: #f0f0f0;
}

.toolbar button.is-active {
  background-color: #e0e0ff;
  color: #333;
}

.toolbar button:disabled {
  color: #ccc;
  cursor: not-allowed;
}

.heading-select {
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #fff;
  cursor: pointer;
}

.content-container {
  flex-grow: 1;
  overflow-y: auto;
  background-color: #fff;
  border-radius: 5px;
  border: 1px solid #ccc;
}
.editor-content-area,
.preview-area {
  height: 100%;
  box-sizing: border-box;
}

.ProseMirror {
  padding: 1rem;
  outline: none;
  height: 100%;
}
.ProseMirror:focus {
  border-color: transparent;
  box-shadow: none;
}
.preview-area {
  padding: 1rem;
  overflow-y: auto;
}
</style>
