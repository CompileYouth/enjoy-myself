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
app1.name // 你访问了 name
app.age = 100;  // 你设置了 age，新的值为100
app2.university // 你访问了 university
app2.major = 'science'  // 你设置了 major，新的值为 science
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

	return this.obj;
```

## 第二步

上面的写法咋一看没啥问题，其实问题挺大的，考虑下面两种情况：

- 如果传入参数对象是一个“比较深”的对象（也就是其属性值也可能是对象）

  ```js
  let obj = {
    a: 1,
    b: 2,
    c: {
      d: 3,
      e: 4
    }
  }
  ```
- 如果设置新的值是一个对象的话，新设置的对象的属性是否能能继续响应 getter 和 setter

  ```js
  let app1 = new Observer({
    name: 'youngwind',
    age: 25
  });

  app1.name = {
    lastName: 'liang',
    firstName: 'shaofeng'
  };

  app1.name.lastName;
  // 这里还需要输出 '你访问了 name' 和 '你访问了 lastName'
  app1.name.firstName = 'lalala';
  // 这里还需要输出 '你设置了firstName, 新的值为 lalala'
  ```

实现：

```js
function Observer(obj) {
	this.obj = obj;
	Object.keys(this.obj).forEach(key => wrapAttr(this.obj, key, this.obj[key]));

	function wrapAttr(obj, key, val) {
		const childObj = observeDeep(val);
		Object.defineProperty(obj, key, {
			enumerable: true,
			configurable: true,
			get: () => {
				console.log(`你访问了 ${key}`);
				return val;
			},
			set: newVal => {
				console.log(`你设置了 ${key}，新的值为${newVal}`);
				val = observeDeep(newVal);
			}
		});
	}

	function observeDeep(val) {
		if (!val || typeof val !== 'object') { return }

		return new Observer(val);
	}

	return this.obj;
}
```

这就完善了嵌套对象的访问。
