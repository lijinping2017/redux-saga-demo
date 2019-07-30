# 基于create-react-app引入redux-saga的脚手架搭建 


## 一、背景说明
`redux-saga` 顾名思义就是在 `redux` 的基础上插入 `saga` 中间件，`saga` 是用来处理异步操作的，它的内部核心是使用叫 `Generator` 函数以同步的方式实现异步操作，因此要实现 `saga` 的效果，首先需要引入 `redux`，由于 `create-react-app` 没有集成 `redux`和 `saga`，因此想要在项目中使用 `redux-saga` ，就要在项目中引入有关 `redux`和`redux-saga` 的插件，具体操作如下：

## 二、操作步骤

### 1、创建项目
(1)、进入到你想创建项目存放的磁盘文件，比如 `D:\test` 文件，双击进入 `test` 文件后，在空白的地方按住 `shift` 键再用鼠标点击右键，选择在**此处打开命令窗口（w)**,此时会弹出一个命令行窗口。

(2)、我们需要使用create-react-app脚手架创建一个项目，此时在命令行窗口中输入

`create-react-app redux-saga-demo`

回车后等待项目安装完成，此时项目名称为 `redux-saga-demo`。

(3)、安装完成后，我们会在 `D:\test` 中看到新增的 `redux-saga-demo` 文件，文件结构如下：

	├── node_modules                  // 模块安装依赖包
	├── public                        //公共文件，可以不用管
	│   ├── favicon.ico               //图标
	│   ├── index.html                //入口html
	│   ├── manifest.json             //manifest配置文件，指定应用名称、图标等信息
	├── src 						  //编写自己代码的存放文件
	│   ├── App.css                   //app的引用css文件
	│   ├── App.js					  //组件js文件
	│   ├── App.test.js               //测试文件
	│   ├── index.css                 //idnex引用的css文件
	│   └── index.js				  //页面入口文件
	│   ├── logo.svg                  //页面图片
	│   ├── serviceWorker.js          //加速程序运行文件
	├──.gitignore                     //提交到远程代码库时要忽略的文件
	├──package.json                   //用来声明项目的各种模块安装信息，脚本信息等
	├──package-lock.json              //用来锁定模块安装版本的，确保安装的模块版本一致
	├──README.md					  //盛放关于这个项目的说明文件 


### 2、启动项目
此时我们要验证一下项目能否运行成功，则在命令行窗口中输入

`cd redux-saga-demo`

进入到项目里面，再输入命令

`npm star`

后，在弹出的浏览器 `localhost://3000` 页面下能看到页面展示出来，如下图：
![react](https://github.com/lijinping2017/redux-saga-demo/raw/master/redux-images/react.jpg)

就说明项目能正常运行成功了。

### 3、安装Redux和saga插件

(1)、首先我们要安装有关 `redux` 的插件，这里需要安装两个有关 `redux` 的插件，分别是 `redux`、`react-redux`，一个 `redux-saga` 插件，由于刚刚启动了项目，服务器在运行中，因此在安装插件之前先结束进程，具体操作是：在命令行窗口中按住 `ctrl+c` 键结束运行，然后在命令行中输入

`npm install redux react-redux redux-saga --save-dev`

回车等待安装完成。

(2)安装完成后，打开 `package.json` 文件，我们会看到 `devDependencies` 多 `redux` 和
`rect-redux` 和 `redux-saga` 三个插件以及插件的版本号信息，如图所示：

![package-json](https://github.com/lijinping2017/redux-saga-demo/raw/master/redux-images/package-json.jpg)

此时项目就可以使用 `redux`、`react-redux`、 `redux-saga` 这三个插件了。


### 4、使用saga的步骤

下面我们通过使用 `saga` 实现异步请求数据，并结合 `redux` 实现将异步请求回来的数据展现在页面上。

##### (1)、创建组件

在 `src` 文件夹下创建一个名为 `components` 的文件夹，用来存放项目的自定义组件，在`components` 文件夹下创建一个 `MySaga.js` 文件，代码如下：

MySaga.js

	import React, { Component } from 'react';
	
	import { connect } from 'react-redux';
	import { asyncAction } from '../actions.js';
	
	class MySaga extends Component {
		constructor(props) {
			super(props);
		}
	
		componentDidMount() {
		    console.log(this.props)
		} 
	
		render() {
			return(
				<div>
					<h2>异步请求回来的数据：</h2>
					<button onClick={this.props.getdata}>点击获取数据</button>
					<ul>
						{
							this.props.dataArray.map((value,key) => {
								return (
									<li key={key}>
										{value}
									</li>
								)
							})
						}
					</ul>
				</div>
			) 
	    
	  	}
	}
	
	const mapStateToProps = (state) => {
		return {
			dataArray: state
		}
	}
	
	const mapDispatchToProps = (dispatch) => {
		return {
			getdata: ()=>dispatch(increaseAction)
		}
	}
	
	export default connect(mapStateToProps,mapDispatchToProps)(MySaga);



##### (2)、编写saga.js

首先在 `src` 文件夹下创建一个 `saga.js` 文件，用于编写 `saga` 异步操作的函数，具体代码如下：

saga.js

	import { takeEvery, call, put, all } from "redux-saga/effects";
	import { AsyncAction, getDataAction } from "./actions";
	import axios from 'axios';
	
	function* watchSaga() {
		yield takeEvery("AsyncData",workSaga);
	}
	
	function axiosData() {
		return  axios.get('https://jsonplaceholder.typicode.com/todos')
	}
	
	function* workSaga() {
		const data = yield call(axiosData);
		yield put(getDataAction(data.data));
	}
	
	export default function* rootSagaMiddle() {
		yield all([
			watchSaga()
		])
	}

`saga` 文件中使用到了 `axios` 发起异步请求，它是一个插件，因此在这之前要先安装 `axios` 这个插件，在命令行窗口中输入

`npm install axios --save-dev`

安装完后就可以在 `saga.js` 文件头部引用 `saga` 了， `takeEvery()` 用来监听所有 `dispath` 出来的 `action`，当 `action.type`  匹配时就触发 `workSaga` ,`call()` 用来发送指令，由 `saga` 中间件去执行获取异步数据请求操作，`put()` 用来派发一个 `action`，用于 `reducer` 接受 `action`，处理 `state`。


#### (3)、编写actions.js、reducer.js

在 `src` 文件夹下创建两个 `js` 文件，分别为 `actions.js` 和 `reducer.js`,具体代码如下：

actions.js
	
	export const asyncAction  = {
		type: "AsyncData"
	}
	
	export function getDataAction(data) {
		return {
			type: "GetData",
			data: data
		}
	}


reducer.js

	import { getDataAction } from './actions.js';
	
	const  reducer = (state = [],action) => {
		switch(action.type) {
			case 'GetData':
				return action.data;
			default:
				return state;
		}
	}
	
	export default reducer;


##### (4)、修改index.js、App.js

此时要将 `reducer` 处理返回的 `state` 值放入 `store` 树上,并使 `saga` 运行起来，以及将达到渲染 `MySaga` 组件，因此要修改 `index.js` 和 `App.js` 文件代码，具体代码如下：

index.js

	import React from 'react';
	import ReactDOM from 'react-dom';
	import App from './App';
	import * as serviceWorker from './serviceWorker';
	import { createStore,applyMiddleware} from "redux";
	import createSagaMiddleware from "redux-saga";
	import reducer from "./reducer.js";
	import { Provider } from "react-redux";
	import rootSagaMiddle from "./saga";
	
	const sagaMiddle = createSagaMiddleware();
	const store = createStore(
		reducer,
		applyMiddleware(sagaMiddle)
	);
	
	sagaMiddle.run(rootSagaMiddle);
	
	
	ReactDOM.render(
		<Provider store={store}>
			<App />
		</Provider>,
		document.getElementById('root')
	);
	
	serviceWorker.unregister();


App.js

	import React from 'react';
	import MySaga from "./components/MySaga";
	
	function App() {
	    return (
	        <div className="App">
	          <MySaga />
	        </div>
	    );
	}
	
	export default App;


### 5、测试redux项目是否成功

到这里使用 `saga` 的步骤基本完成了，这时我们需要在浏览器中查看我们编写的代码是否在页面上生效，因为需要重新启动项目，在命令行窗口中输入

 `npm start`

启动项目后，会弹出新的浏览器 `localhost://3000` 页面，此时页面是这样的：
![saga](https://github.com/lijinping2017/redux-saga-demo/raw/master/redux-images/saga.jpg)

出现这样的页面就说明项目没有错误，达到了我们想要的效果，这时要验证一下点击**点击获取数据**按钮时，是否在页面中将异步数据展示出来，测试结果如下：

点击**点击获取数据**按钮：
![getdata](https://github.com/lijinping2017/redux-saga-demo/raw/master/redux-images/saga-getdata.jpg)


好了，此时的页面效果可以说明了我们已经完成了基于 `create-react-app` 引入 `saga` 来实现异步获取数据的效果了。




