#使用Mocha进行 Restful API 测试


简单来说mocha是一个测试框架,可以在这个框架下使用各种断言库(断言就是作出一个预测,不符合则抛出),
mocha支持BDD和DTT有不同风格的


兼容express,

###工具选用
* mocha - 测试框架
* supertest - 请求工具
* should - 断言库


###环境搭建
- 第一步安装环境 `npm install --save-dev mocha supertest` ,这样就在package.json生成了一个"devDependencies".
- 进入项目目录 `mkdir test && cd test && touch test.js`,mocha默认会执行test下所有js后缀的文件.

###原理和代码实现
1. 用编辑器打开上一步新建的`test.js` ,
先写请求(mocha全局安装?)
```var supertest = require("supertest");
```var should = require("should");

2. 希望
     it(‘s
     hould dosomthing’, function(done) {
       doSomething(done);
       // 下面的代码只有当done在doSomething中被异步调用之后才会运行
     });






- mocha 的基本框架,这里只写api测试的部分,只写一个post用例,网上有大量单元测试的例子,此处不做介绍(也因此被坑), before,after,ech,after.

- 格式是一个describe介绍一下,function()里可以嵌套多个it,理解成目录,it风格同上,
done(坑),
server(上文中的请求,也可用express);

下面的内容跟伪代码几乎一样就不介绍了,其中should尤甚,提供了各种比如be it 之类的无意义,此处省略,





###优化(提升逼格环节)

1. 简单复用
2. 目前api内容较少,后续自动化测试可以考虑把api独立到excel/xml文件中.
3. 启动问题  使用make




    
//没改好
  it.('just a test', function(done){
    testGet( , done);
  });




  var testGet = function (uri, json, equal) {
    server
    .post(uri)
    .send(json)
    .expect(200)
    .end(function(err,res){
      res.status.should.equal(200);
      //res.body.error.should.equal(false);
      //res.body.data.should.equal(30);
      res.body.state.should.equal(equal);
        done();
      });
  };









###坑


- 断言库遇到错误即会抛出并停止执行,所以很多东西要分开写.
- done,执行异步代码时需要在结束时调用回调函数.
对异步理解不清楚,最后日志打印到done()的上一步了才发现问题,最后搞了一个上午,让我感觉很对不起工资.教训就是步子小一点不要掺入过多干扰因素.
- 关键字
- mocha 默认2秒超时,运行时可以根据需求加 -t 2000 修改,更多参数 --help查看.

###最后
就我的理解测试最重要的是如何设计用例,求各位大神分享.

###参考连接
更多内容推荐使用关键字 `Mocha Restful API test` 搜索.


http://ju.outofmemory.cn/entry/86908
