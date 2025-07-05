<template>
  <div class="toc-container">
    <h2>目录</h2>
    <p class="placeholder-text">
      文档的目录结构将在这里自动生成。
    </p>
    <!-- 之后可以填充动态生成的目录列表 -->
    <ul>
      <li>介绍</li>
      <li>第一
        <ul>
          <li>第一节</li>
        </ul>
      </li>
      <li>第二章</li>
    </ul>
  </div>
</template>

<script setup>
// 目前不需要脚本逻辑
</script>

<style scoped>
.toc-container {
  padding: 20px;
  background-color: #ffffff;
  border-right: 1px solid #e0e0e0;
  height: 100vh; /* 撑满整个视口高度 */
  overflow-y: auto; /* 如果内容过多，则显示滚动条 */
  box-sizing: border-box; /* 确保 padding 不会增加宽度 */
}

h2 {
  margin-top: 0;
  color: #333;
  border-bottom: 2px solid #f0f0f0;
  padding-bottom: 10px;
}

.placeholder-text {
  color: #888;
  font-size: 0.9em;
  font-style: italic;
}

ul {
  padding-left: 20px;
  list-style-type: none;
}

li {
  margin: 10px 0;
  color: #555;
  cursor: pointer;
}

li:hover {
  color: #007bff;
}
</style>
