# express-jade-compiled
 [jade](https://github.com/jadejs/jade) 模版服务器端编译，前端使用。

它是一个 [express](https://github.com/strongloop/express) 中间件,需求express 4.x.

[Document in English](https://github.com/hezedu/express-jade-compiled/blob/master/README.en.md)
###install
```
npm install express-jade-compiled
```
##API
###jade_compiled(path, options);
例：
```js
var jade_compiled = require('express-jade-compiled');
//简单
app.use('/jade_compiled',jade_compiled(path.join(__dirname, 'jade_compiled')));

```
### Options
- `maxAge` 模版文件的maxAge，默认0。
- `watch` 是否监测文件修改,动态更新缓存. 默认开发环境为true, 线上为false;
- `uglify` 是否压缩编译文件, 默认线上环境为true, 开发环境为false;
- `wrap` 包装编译好的文件,用`$COMPILED$`代表编译后的字符串。默认是AMD规范：`"define(function(){return $COMPILED$})"`


##特性：//-COMPONENT
新加了`//-COMPONENT`标识来分割模版，以达到多重利用的目地。

####例:
jade未加//-COMPONENT
```jade
h1 hello
h1 world
```
编译后(format后)：
```js
define(function() {
  return function tpl(locals) {
    var buf = [];
    var jade_mixins = {};
    var jade_interp;
    buf.push("<h1>hello</h1><h1>world</h1>");;
    return buf.join("");
  }
})
```

而jade加上//-COMPONENT
```jade
//-COMPONENT hello
h1 hello
//-COMPONENT world
h1 world
```
编译后(format后)：
```js
define(function() {
  return {
    "hello": function tpl1(locals) {
      var buf = [];
      var jade_mixins = {};
      var jade_interp;
      buf.push("<h1>hello</h1>");;
      return buf.join("");
    },
    "world": function tpl2(locals) {
      var buf = [];
      var jade_mixins = {};
      var jade_interp;
      buf.push("<h1>world</h1>");;
      return buf.join("");
    }
  }
})
```
####jade runtime.js

前端需要引用Jade的runtime.js，本项目已压缩好了一个(3KB),并提供了一个静态属性：`jade_runtime_min_path`你可以:
```js
app.use('/jade_runtime_min.js', express.static(jade_compiled.jade_runtime_min_path));
```
##客户端引用示例
```html
<!DOCTYPE html>
<html>
  <head>
    <script src="/jade_runtime_min.js"></script>
    <script src="require.js"></script> <!--默认wrap：AMD，所以此处引了require -->
  </head>

  <body>
    <script type="text/javascript">
      require(['/jade_compiled/some.jade'], function(tpl){
        document.write(tpl());
        //or
        //document.write(tpl.main());
      })
    </script>
  </body>
</html>
```
##QA
1.为什么我修改了模版，但没有生效？

答：这一定是**文件名大小写**与请求URL不一致导致的。




