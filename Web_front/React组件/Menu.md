# 一、React 测试代码

```js
// menu_test.js

import React, { useState } from "react";
import Menu, { Item as MenuItem, SubMenu } from "rc-menu";
import "rc-menu/assets/index.css"; // 引入默认样式

const WorkingMenu = () => {
  // useState 调用后会返回一个 长度为 2 的数组:第一个为变量，第二个为函数，[] 是初始化的值
  const [openKeys, setOpenKeys] = useState([]); // 控制展开的子菜单

  // 处理子菜单展开/收起
  const onOpenChange = (newOpenKeys) => {
    console.log("Clicked key2:", newOpenKeys);
    // 只保持一个子菜单展开
    const latestOpenKey = newOpenKeys.find((key) => !openKeys.includes(key));
    setOpenKeys(latestOpenKey ? [latestOpenKey] : []);
  };

  // 菜单项点击处理
  const handleClick = (info) => {
    console.log("Clicked menu item:", info.key);
  };

  // 在rc-menu库中，onClick、openKeys和onOpenChange确实是Menu组件自带的属性，它们是该组件API的一部分
  // onClick： 菜单项点击事件处理函数
  // openKets：控制子菜单哪些处于展开状态
  // onOpenChange：当子菜单展开状态发生变化时触发的回调函数
  return (
    <Menu
      onClick={handleClick}
      openKeys={openKeys}
      onOpenChange={(keys) => onOpenChange(keys)}
      mode="inline" // 垂直模式支持展开
    >
      <MenuItem key="1">首页</MenuItem>
      <MenuItem key="2">产品</MenuItem>
      <SubMenu key="sub1" title="服务">
        <MenuItem key="3">设计服务</MenuItem>
        <MenuItem key="4">开发服务</MenuItem>
      </SubMenu>
      <MenuItem key="5">联系我们</MenuItem>
    </Menu>
  );
};

export default WorkingMenu;

```

```js
// App.js
export default function MyApp() {
  return <BasicMenu></BasicMenu>;
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return <button onClick={onClick}>Clicked {count} times</button>;
}

```

```js
// index.js

// index.js 是在文件中创建的组件和 Web 浏览器之间的桥梁。App.js

// 导入React库
import React, { StrictMode } from "react";

// React dom
import { createRoot } from "react-dom/client";

// Css 文件
import "./styles.css";

// 组件的概念
// export default function Game(), 所以默认会去调用Game 函数
import App from "./App";

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```css
// styless.css

* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

# 二、组件相关信息

- https://github.com/react-component/menu