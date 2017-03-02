自己写个 JSONP。

做过跨域调服务的人恐怕对 JSONP 都不陌生，因为浏览器出于安全考虑，禁止从 A 域调 B 域的服务，所以当需要跨域调服务时就会出现问题。但写过网页的都知道，script 标签中的 JS 文件如果是第三方的话，文件所在域一般是与当前域不一样的，也就是说用 script 标签加载其他域中的文件是不会有问题的，所以我们就从这里入手，自己动手写一个 JSONP。

```js
function jsonp(url, params, callback) {
  const parsmQuery = [];
  Object.keys(params).forEach(key => {
    parsmQuery.push(`${encodeURIComponent(key)}=${encodeURIComponent(params[key])}`);
  })
  let _url;
  _url = url + (parsmQuery.length > 0 ? `?${parsmQuery.join('&')}` : `?`);
  _url = _url + `&callback=callback2017`;

  console.log(_url);

  const script = document.createElement('script');
  script.setAttribute('src', _url);
  script.type = "text/javascript";
  document.getElementsByTagName('head')[0].appendChild(script);

  window['callback2017'] = (res) => {
    callback(res);
  }
}

jsonp('http://music.163.com/api/user/playlist/', {
  uid: 40652589,
  limit: 1,
  offset: 0
}, function(data) {console.log(data)})
```
