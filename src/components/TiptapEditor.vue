<template>
  <div class="editor-wrapper">
    <div v-if="editor" class="toolbar">
      <button @click="togglePreview" :class="{ 'is-active': isPreviewMode }">
        {{ isPreviewMode ? '返回编辑' : '预览' }}
      </button>

      <template v-if="!isPreviewMode">
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
      </template>
    </div>

    <div class="content-container">
      <editor-content v-show="!isPreviewMode" :editor="editor" class="editor-content-area" />

      <div v-if="isPreviewMode" v-html="previewContent" class="preview-area ProseMirror"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted, computed } from 'vue';
import { useEditor, EditorContent } from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';
import Collaboration from '@tiptap/extension-collaboration';
import CollaborationCursor from '@tiptap/extension-collaboration-cursor';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

const isPreviewMode = ref(false);

const ydoc = new Y.Doc();
const roomName = 'tiptap-demo-room';
const websocketUrl = 'ws://localhost:1234';
const provider = new WebsocketProvider(websocketUrl, roomName, ydoc);

const userColor = '#' + Math.floor(Math.random()*16777215).toString(16).padStart(6, '0');
const userName = 'User ' + Math.floor(Math.random() * 100);

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

const previewContent = computed(() => {
  return editor.value ? editor.value.getHTML() : '';
});

const togglePreview = () => {
  isPreviewMode.value = !isPreviewMode.value;
};

onUnmounted(() => {
  if (provider) provider.destroy();
  if (ydoc) ydoc.destroy();
});
</script>

<style>
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
  gap: 8px;
  margin-bottom: 16px;
  padding: 8px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 5px;
  flex-shrink: 0;
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
  background-color: #007bff;
  color: #fff;
  border-color: #007bff;
}

.content-container {
  flex-grow: 1;
  overflow-y: auto;
  background-color: #fff;
  border-radius: 5px;
  border: 1px solid #ccc;
  position: relative;
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
