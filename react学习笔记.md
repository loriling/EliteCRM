# JSX
1. 表达式: {expr}
2. 属性: <div id={expr}>
3. jsx也是表达式: <p>{jsx}</p>


###1. PureComponent 纯组件，只关展示，没有逻辑
设计思想：状态的控制尽量放在一个隔离的范围内，展示的东西单独去做.
如果数据不改变，就不重复执行render，提高性能。（类似于shouldComponentUpdate里自己判断属性是否改变）
shallowCompare 浅比较

```js
function Comment({ data }) {
	return (<div>
			<p>{data.body}</p>
			<p>{data.author}</p>
		</div>
	);
}

15.3.0之后
class Comment extends React.PureComponent {
	render() {
		return (<div>
				<p>{this.props.body}</p>
				<p>{this.props.author}</p>
			</div>
		);
	}
}

16.6.0之后 memo方法，可以函数式的去写PureComponent
const Comment = React.memo(function(props) {
	return (<div>
			<p>{props.body}</p>
			<p>{props.author}</p>
		</div>
	);
});
```

###2.高阶组件
ES7中的装饰器语法
npm install --save-dev babel-plugin-transform-decorators-legacy

```
const {injectBabelPlugin} = require('react-app-rewired');

module.exports = funciton override(config) {
	config = injectBabelPlugin(
		['import', { libraryName: 'antd', libraryDirectory: 'es', style: 'css'}],
		config,
	)
	config = injectBabelPlugin(
		['@babel/plugin-proposal-decorators', {'legacy': true}],
		config
	)
	return config
}


const withLog = Comp => {
	console.log(Comp.name + ' rendered');
	return props => <Comp {...props} />
};

@withLog
class XXX extends Component {
    render() {
		return (
			<div> ...
			</div>
		)
    }
}

```

###3.组件复合
复合组件给与你足够的敏捷去定义自定义组件的外观和行为，而且是以一种明确和安全的方式进行，如果组件间有公用的非UI逻辑，讲它们抽取为JS模块导入使用而不是继承它。

###4.children是什么？答：任何js表达式
```
const Api = () {
	getUser() {
		return { name: "lori", age: 35 }
	}
}

function Fetcher(props) {
	const user = Api[props.name]();
	return props.children(user)
}

export default function() {
	return (
		<div>
			<Fetcher name="getUser">
				{({ name, age }) => {
					<p>{name} - {age}</p>
				}}
			</Fetcher>
		</div>
	)
}
```

```
function RadioGroup(props) {
	return <div>
		{React.Children.map(props.children, child => {
			// vdom不可更改，克隆一个新的去修改才行
			// child.props.name = props.name;
			return React.cloneElement(child, {name: props.name}; 
		})}
	</div>;
}
function Radio({children, ...rest}) {
	return (
		<label>
			<input type="radio" {...rest} />
			{children}
		</label>
	)
}

<RadioGroup name="mvvm">
	<Radio value="vue">vue</Radio>
	<Radio value="react">react</Radio>
</RadioGroup>
```