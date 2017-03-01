MVC 模型中动态绑定的自我实现。

## 第一步

实现一个 Observer 满足下面的条件：

```js
let app1 = new Observer({
  name: 'youngwind',
  age: 25
});

let app2 = new Observer({
  university: 'bupt',
  major: 'computer'
});

// 要实现的结果如下：
app1.data.name // 你访问了 name
app.data.age = 100;  // 你设置了 age，新的值为100
app2.data.university // 你访问了 university
app2.data.major = 'science'  // 你设置了 major，新的值为 science
```

实现：

```js
function Observer(obj) {
	this.obj = obj;
	Object.keys(this.obj).forEach(key => wrapAttr(this.obj, key, this.obj[key]));

	function wrapAttr(obj, key, val) {
		Object.defineProperty(obj, key, {
			enumerable: true,
			configurable: true,
			get: () => {
				console.log(`你访问了 ${key}`);
				return val;
			},
			set: newVal => {
				console.log(`你设置了 ${key}，新的值为${newVal}`);
				val = newVal;
			}
		});
	}

	return {
		data: this.obj
	};
}
```
