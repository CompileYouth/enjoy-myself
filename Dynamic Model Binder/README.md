MVC 模型中动态绑定的自我实现。

## 第一步

实现一个 Observer 满足下面的条件：

```js
const app1 = new Observer({
  name: 'youngwind',
  age: 25
});

const app2 = new Observer({
  university: 'bupt',
  major: 'computer'
});

// 要实现的结果如下：
app1.data.name // 你访问了 name
app1.data.age = 100;  // 你设置了 age，新的值为100
app2.data.university // 你访问了 university
app2.data.major = 'science'  // 你设置了 major，新的值为 science
```

实现：

```js
function Observer(data) {
	this.data = data;
	Object.keys(this.data).forEach(key => wrapAttr(this.data, key, this.data[key]));

	function wrapAttr(data, key, val) {
		Object.defineProperty(data, key, {
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
}
```

## 第二步

上面的写法咋一看没啥问题，其实问题挺大的，考虑下面两种情况：

- 如果传入参数对象是一个“比较深”的对象（也就是其属性值也可能是对象）

  ```js
  const obj = {
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
  const app1 = new Observer({
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
function Observer(data) {
  this.data = data;
  this.walk(data)
}

Observer.prototype.walk = function(obj) {
  let val;
  Object.keys(this.data).forEach(key => {
    val = this.data[key];
    if(typeof val === 'object') {
      new Observer(val);
    }
    this.convert(key, val);
  });
}

Observer.prototype.convert = function (key, val) {
    Object.defineProperty(this.data, key, {
        enumerable: true,
        configurable: true,
        get: function () {
            console.log('你访问了' + key);
            return val;
        },
        set: function (newVal) {
            console.log('你设置了' + key);
            console.log('新的' + key + ' = ' + newVal)
            if (newVal === val) return;
            val = newVal;
        }
    })
};
```

这就完善了嵌套对象的访问。
