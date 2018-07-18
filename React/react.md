* Virtual DOM
	DOM Document Object Model 文档对象模型
	将网页内所有内容映射到一颗树形结构的层级对象模型上，浏览器提供对DOM的支持，用户可以用脚本调用DOM API来动态的修改DOM结点，从而达到修改网页的目的，这种修改是浏览器中完成的，浏览器会根据DOM的改变重绘改变的DOM节点部分

	修改DOM重新渲染代价太高，为了减少DOM的重绘，提出了Virtual DOM，所有的修改都是在Virtral DOM中完成的，通过比较算法，找出浏览器DOM之间的差异，使用这个差异来操作DOM，浏览器只需要渲染改变的部分即可。

	React实现了DOM Diff算法，可以高效比对Virtual DOM和DOM的差异

```js  

#导入模块
import React from 'react';
import ReactDom from 'react-dom'
import xxx from './*'

#组件类定义，从React.Component类上继承，这个类生成JSXElement对象，即React元素
class Root extends React.Component

#渲染函数，返回组件中渲染的内容，只能返回唯一一个顶级元素
render()

# 第一个参数是JSXElement对象，第二个是DOM的Element元素，将React元素添加到DOM的Element元素中并渲染
ReactDom.Render(<Root/>,document.getElementById('root'));

###axios

####get


####post


```

* JSX规范
	* 首字符小写是html标签，首字母大写为组件
	* 要求所有的标签必须闭合，br应该写为<br />， /前留有一个空格
	* 单行省略小括号，多行使用小括号
	* 元素有嵌套，建议多行，注意缩进
	* JSX表达式：使用{},如果大括号内使用了引号，会当做字符串处理

* 组件状态-state *
	* 每个React组件都有一个状态变量state,是一个js对象，可以为它定义属性来保存需要的值
	* 只有该state状态变化了，才会触发DOM重新渲染
	* state是组件内部使用的，为组件的私有属性
	* 使用setState方法更新state状态
	* 正在更新中的state不可以使用setState
