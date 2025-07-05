<template>
  <div id="main-layout">
    <!-- 左侧目录区 -->
    <div class="sidebar">
      <TableOfContents />
    </div>

    <!-- 右侧编辑区 -->
    <div class="main-content">
      <TiptapEditor />
    </div>
  </div>
</template>
<script>
import TableOfContents from './components/TableOfContents.vue';
import TiptapEditor from './components/TiptapEditor.vue';
export default {
  name: 'App',
  components: {
    TableOfContents,
    TiptapEditor
  }
}
</script>

<style>
/* 全局基础样式 */
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  background-color: #f0f2f5;
}

/* 主布局容器 */
#main-layout {
  display: flex;
  width: 100vw;
  height: 100vh;
}

/* 左侧边栏 (目录) */
.sidebar {
  flex: 0 0 30%; /* 不伸缩，不收缩，基础宽度为 30% */
  max-width: 400px; /* 可以设置一个最大宽度 */
  min-width: 250px; /* 可以设置一个最小宽度 */
}

/* 右侧主内容区 (编辑器) */
.main-content {
  flex: 1; /* 占据剩余的所有空间 */
  overflow: hidden; /* 防止内部元素溢出影响布局 */
}
</style>
