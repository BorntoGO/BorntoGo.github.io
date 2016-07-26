---
layout: post
title: 使用 mocha 进行 RESTful API 测试
category: [testing, node.js]
tags: [mocha, RESTful, testing, node.js]
---

简单来说mocha是一个测试框架,支持BDD和TDD不同风格的接口,支持before,after等前/后置条件.可以在这个框架下使用各种断言库(断言就是作出一个预测,不符合则抛出,mocha就是整合这些断言结果).

### 工具选用
* mocha - 测试框架
* supertest - 请求工具
* should - 断言库


### 环境搭建
- 第一步安装环境 `npm install --save-dev mocha supertest`.
- 进入项目目录 `mkdir test && cd test && touch test.js`, mocha默认会执行test下所有js后缀的文件.

### 代码实现
在上一步新建的test.js中写入:

```
var supertest = require("supertest");
var should = require("should");
var request = supertest.agent("http://192.168.0.120:3000");

describe("Restful APIs test - GET",function(){
  it("广场列表推荐-获取热门标签",function(done){
    request
      .get("/api/v1/recommend/tags?pageSize=4")
      .expect(200)
      .end(function (err,res) {
        res.status.should.equal(200);
        res.body.state.should.equal(0);
        done();
      });
  });
});
```

解读：

- `describe (moduleName, testDetails)`
describe modueName描述场景,可以把describe()当成目录理解.
- `it (info, function)`
info描述测试用例,一般简要说明预期结果,具体的测试语句放在it的回调函数里.
- `expect()`
除了请求状态码,还可以请求body,自己写断言,或者几种混合.
- `res.body.state.should.equal(0)`
should断言返回的body中的state字段结果是否等于0(本项目正常返回时body中此字段值为0,此处用来简单举例).另外should还提供了a,an,it,have等帮助理解的无意义链条函数.
- `done()`
为异步方法完成回调(此坑已踩),执行异步代码时需要在结束时调用回调函数.

### 简单复用
可以把it部分拿过来做一个简单复用

```
describe('Restful APIs test - GET', function (){

  var testGet = function (uri, state, done) {
    console.log('       \n', uri);
    request
    .get(uri)
    .expect('Content-type',/json/)
    .end(function (err, res) {
      res.body.should.have.property('msg');
      res.status.should.equal(200);
      res.body.msg.should.exist;
      res.body.result.should.exist;
      res.body.state.should.equal(state);
      res.body.result.length.should.below(100);
      res.body.result.should.be.Array();
      done (err);
    });
  };  


  var list = [-1, 0, 15, 16, '*&@!'];
  _.each(list,function(e){
    var pageSize = e;
    _.each(list, function (e) {
      it("广场列表推荐", function(done){
      var pageId = e;
      testGet(api.concat('pageId=').concat(pageId)
        .concat('&pageSize=').concat(pageSize), 0, done);
      });
    });
  });
});
```


工具使用就是这些了,后面更重要的是用例的设计.

### 单元测试

SuperTest 可以直接驱动express,用来做单元测试超级方便的.
测试统计,单元测试时可以使用istanbul这个,会显示覆盖率等各种信息,并可以生成html.
另外还有各种高大上的工具可用,其实各位后端同学码完代码完全可以顺手做做单元测试哒 ~

### 参考连接

<http://ju.outofmemory.cn/entry/86908>
<https://www.npmjs.com/package/supertest>
<http://www.moye.me/2014/11/22/bdd_mocha/>



