# mongoose-introduction
mongoose主要知识点的总结


     ![mongoose](https://github.com/GIScyw/mongoose-introduction/blob/master/1.png)




## Mongoose 述

Mongoose里面最重要的三个概念是Schemas，models和Documents，另外还有验证，中间件对前面三个至关重要，这里像填充Populate和鉴别器Discriminators的相关知识就不介绍了，平时基本上不太能用到，具体的可以查阅官方文档。


Mongoose的一切始于Schema，Schema定义collection里的文档的构成，models是从Schema编译来的构造函数，documents是models的实例，它可以从数据库保存和读取数据

```javascript
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);

var small = new Tank({ size: 'small' });
small.save(function (err) {
  if (err) return handleError(err);
  // saved!
})

// or

Tank.create({ size: 'small' }, function (err, small) {
  if (err) return handleError(err);
  // saved!
})
```

### 模式Schemas

**1. 如何为models实例（Documents）定义我们自己的方法**
虽然douments有很多自带的实例方法，但我们也可以自定义我们自己的方法,这时需要在schema上定义方法。

```javascript
  var animalSchema = new Schema({ name: String, type: String });

  animalSchema.methods.findSimilarTypes = function(cb) {
    return this.model('Animal').find({ type: this.type }, cb);
  };
  
  var Animal = mongoose.model('Animal', animalSchema);
  var dog = new Animal({ type: 'dog' });

  dog.findSimilarTypes(function(err, dogs) {
    console.log(dogs); // woof
  });
```

也可以添加静态方法

```javascript
 animalSchema.statics.findByName = function(name, cb) {
    return this.find({ name: new RegExp(name, 'i') }, cb);
  };

  var Animal = mongoose.model('Animal', animalSchema);
  Animal.findByName('fido', function(err, animals) {
    console.log(animals);
  });
```

**2.Schemas有很多的配置选项，可以在构造时或者直接set**

```javascript
new Schema({..}, options);

// or

var schema = new Schema({..});
schema.set(option, value);
```

### 模型Models

#### 查询

查询文档可以用model的find，findById,findOne和where这些静态方法

#### 删除

remove方法

#### 更新

model的update方法可以修改数据库中的文档，不过不会把文档返回

findOneAndUpdate方法用来更新单独一条文档并且返回给应用层

### 文档Documents

#### 检索

检索方法较多，暂不阐述

#### 更新

更新的方法也比较多

- findById+改变值得操作

  ```javascript
  Tank.findById(id, function (err, tank) {
    if (err) return handleError(err);
  
    tank.size = 'large';//或者使用tank.set({ size: 'large' });
    tank.save(function (err, updatedTank) {
      if (err) return handleError(err);
      res.send(updatedTank);
    });
  });
  ```

- update

  ```javascript
  Tank.update({ _id: id }, { $set: { size: 'large' }}, callback);
  ```

- findByIdAndUpdate
  这个方法会返回文档

  ```javascript
  Tank.findByIdAndUpdate(id, { $set: { size: 'large' }}, { new: true }, function (err, tank) {
    if (err) return handleError(err);
    res.send(tank);
  });
  ```

- 其他方法如findAndUpdate/Remove查找并返回最多一个文档

#### 验证

Document会在被保存之前验证

#### 覆盖

可以用.set( )覆盖整个文档

```javascript
Tank.findById(id, function (err, tank) {
  if (err) return handleError(err);
  // Now `otherTank` is a copy of `tank`
  otherTank.set(tank);
});
```

### 查询queries

[Model](https://cn.mongoosedoc.top/docs/api.html#model_Model) 的方法中包含查询条件参数的（ [find](https://cn.mongoosedoc.top/docs/api.html#model_Model.find) [findById](https://cn.mongoosedoc.top/docs/api.html#model_Model.findById) [count](https://cn.mongoosedoc.top/docs/api.html#model_Model.count) [update](https://cn.mongoosedoc.top/docs/api.html#model_Model.update) ）都可以按以下两种方式执行：

- 传入 `callback` 参数，操作会被立即执行，查询结果被传给回调函数（ callback ）。

- 不传 `callback` 参数，[Query](https://cn.mongoosedoc.top/docs/api.html#query-js) 的一个实例（一个 query 对象）被返回，这个 query 提供了构建查询器的特殊接口。[Query](https://cn.mongoosedoc.top/docs/api.html#query-js) 实例有一个 `.then()` 函数，用法类似 promise。

  传callback参数的情况：

```javascript
var Person = mongoose.model('Person', yourSchema);

// 查询每个 last name 是 'Ghost' 的 person， select `name` 和 `occupation` 字段
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
  // Prints "Space Ghost is a talk show host".
  console.log('%s %s is a %s.', person.name.first, person.name.last,
    person.occupation);
});
```

不传callback参数的情况：

```javascript
// 查询每个 last name 是 'Ghost' 的 person
var query = Person.findOne({ 'name.last': 'Ghost' });

// select `name` 和 `occupation` 字段
query.select('name occupation');

// 然后执行查询
query.exec(function (err, person) {
  if (err) return handleError(err);
  // Prints "Space Ghost is a talk show host."
  console.log('%s %s is a %s.', person.name.first, person.name.last,
    person.occupation);
});
```

### 模式类型SchemaTypes

有一下SchemaTypes:

- String

- Number

- Date

- Buffer

- Boolean

- Mixed

- ObjectId

- Array

- Decimal128
  可以声明schema type为某一种type，或者赋值一个含有type属性的对象

  ```javascript
  var schema1 = new Schema({
    test: String // `test` is a path of type String
  });
  
  var schema2 = new Schema({
    test: { type: String } // `test` is a path of type string
  });
  ```

  除了type属性，以下有一些全部type可用的选项和一些限定部分type使用的选项

  ##### 全部可用

  - `required`: 布尔值或函数 如果值为真，为此属性添加 [required 验证器](https://cn.mongoosedoc.top/docs/validation.html#built-in-validators)
  - `default`: 任何值或函数 设置此路径默认值。如果是函数，函数返回值为默认值
  - `select`: 布尔值 指定 query 的默认 [projections](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/)
  - `validate`: 函数 adds a [validator function](https://cn.mongoosedoc.top/docs/validation.html#built-in-validators) for this property
  - `get`: 函数 使用 [`Object.defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 定义自定义 getter
  - `set`: 函数 使用 [`Object.defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 定义自定义 setter
  - `alias`: 字符串 仅mongoose >= 4.10.0。 为该字段路径定义[虚拟值](https://cn.mongoosedoc.top/docs/guide.html#virtuals) gets/sets

  ```javascript
  var numberSchema = new Schema({
    integerOnly: {
      type: Number,
      get: v => Math.round(v),
      set: v => Math.round(v),
      alias: 'i'
    }
  });
  
  var Number = mongoose.model('Number', numberSchema);
  
  var doc = new Number();
  doc.integerOnly = 2.001;
  doc.integerOnly; // 2
  doc.i; // 2
  doc.i = 3.001;
  doc.integerOnly; // 3
  doc.i; // 3
  ```

  ##### 索引相关

  - `index`: 布尔值 是否对这个属性创建[索引](https://docs.mongodb.com/manual/indexes/)
  - `unique`: 布尔值 是否对这个属性创建[唯一索引](https://docs.mongodb.com/manual/core/index-unique/)
  - `sparse`: 布尔值 是否对这个属性创建[稀疏索引](https://docs.mongodb.com/manual/core/index-sparse/)

```javascript
var schema2 = new Schema({
  test: {
    type: String,
    index: true,
    unique: true // Unique index. If you specify `unique: true`
    // specifying `index: true` is optional if you do `unique: true`
  }
});
```

##### String

- `lowercase`: 布尔值 是否在保存前对此值调用 `.toLowerCase()`
- `uppercase`: 布尔值 是否在保存前对此值调用 `.toUpperCase()`
- `trim`: 布尔值 是否在保存前对此值调用 `.trim()`
- `match`: 正则表达式 创建[验证器](https://cn.mongoosedoc.top/docs/validation.html)检查这个值是否匹配给定正则表达式
- `enum`: 数组 创建[验证器](https://cn.mongoosedoc.top/docs/validation.html)检查这个值是否包含于给定数组

##### Number

- `min`: 数值 创建[验证器](https://cn.mongoosedoc.top/docs/validation.html)检查属性是否大于或等于该值
- `max`: 数值 创建[验证器](https://cn.mongoosedoc.top/docs/validation.html)检查属性是否小于或等于该值

##### Date

- `min`: Date
- `max`: Date

### 验证validation

- 验证定义于 [SchemaType](https://cn.mongoosedoc.top/docs/schematypes.html)
- 验证是一个[中间件](https://cn.mongoosedoc.top/docs/middleware.html)。它默认作为 pre('save')` 钩子注册在 schema 上
- 你可以使用 `doc.validate(callback)` 或 `doc.validateSync()` 手动验证
- 验证器不对未定义的值进行验证，唯一例外是 [`required` 验证器](https://cn.mongoosedoc.top/docs/api.html#schematype_SchemaType-required)
- 验证是异步递归的。当你调用 [Model#save](https://cn.mongoosedoc.top/docs/api.html#model_Model-save)，子文档验证也会执行，出错的话 [Model#save](https://cn.mongoosedoc.top/docs/api.html#model_Model-save) 回调会接收错误
- 验证是可定制的

```javascript
    var schema = new Schema({
      name: {
        type: String,
        required: true
      }
    });
    var Cat = db.model('Cat', schema);

    // This cat has no name :(
    var cat = new Cat();
    cat.save(function(error) {
      assert.equal(error.errors['name'].message,
        'Path `name` is required.');

      error = cat.validateSync();
      assert.equal(error.errors['name'].message,
        'Path `name` is required.');
    });
```

#### 内建validators

Mongoose有一些内建验证器

- 所有 [SchemaTypes](https://cn.mongoosedoc.top/docs/schematypes.html) 都有内建 [required](https://cn.mongoosedoc.top/docs/api.html#schematype_SchemaType-required) 验证器。required 验证器使用 [`checkRequired()` 函数](https://cn.mongoosedoc.top/docs/api.html#schematype_SchemaType-checkRequired) 判定这个值是否满足 required 验证器
- [Numbers](https://cn.mongoosedoc.top/docs/api.html#schema-number-js) 有 [min](https://cn.mongoosedoc.top/docs/api.html#schema_number_SchemaNumber-min) 和 [max](https://cn.mongoosedoc.top/docs/api.html#schema_number_SchemaNumber-max) 验证器.
- [Strings](https://cn.mongoosedoc.top/docs/api.html#schema-string-js) 有 [enum](https://cn.mongoosedoc.top/docs/api.html#schema_string_SchemaString-enum)、 [match](https://cn.mongoosedoc.top/docs/api.html#schema_string_SchemaString-match)、 [maxlength](https://cn.mongoosedoc.top/docs/api.html#schema_string_SchemaString-maxlength) 和 [minlength](https://cn.mongoosedoc.top/docs/api.html#schema_string_SchemaString-minlength) 验证器

#### 自定义验证器

自定义验证器通过传入一个检验函数来定义

```javascript
 var userSchema = new Schema({
      phone: {
        type: String,
        validate: {
          validator: function(v) {
            return /\d{3}-\d{3}-\d{4}/.test(v);
          },
          message: '{VALUE} is not a valid phone number!'
        },
        required: [true, 'User phone number required']
      }
    });

    var User = db.model('user', userSchema);
    var user = new User();
    var error;

    user.phone = '555.0123';
    error = user.validateSync();
    assert.equal(error.errors['phone'].message,
      '555.0123 is not a valid phone number!');

    user.phone = '';
    error = user.validateSync();
    assert.equal(error.errors['phone'].message,
      'User phone number required');

    user.phone = '201-555-0123';
    // Validation succeeds! Phone number is defined
    // and fits `DDD-DDD-DDDD`
    error = user.validateSync();
    assert.equal(error, null);
```

#### 异步自定义验证器

自定义检验器可以是异步的。如果检验函数 返回 promise (像 `async` 函数)， mongoose 将会等待该 promise 完成。 如果你更喜欢使用回调函数，设置 `isAsync` 选项， mongoose 会将回调函数作为验证函数的第二个参数。

```javascript
var userSchema = new Schema({
      name: {
        type: String,
        // You can also make a validator async by returning a promise. If you
        // return a promise, do **not** specify the `isAsync` option.
        validate: function(v) {
          return new Promise(function(resolve, reject) {
            setTimeout(function() {
              resolve(false);
            }, 5);
          });
        }
      },
      phone: {
        type: String,
        validate: {
          isAsync: true,
          validator: function(v, cb) {
            setTimeout(function() {
              var phoneRegex = /\d{3}-\d{3}-\d{4}/;
              var msg = v + ' is not a valid phone number!';
              // 第一个参数是布尔值，代表验证结果
              // 第二个参数是报错信息
              cb(phoneRegex.test(v), msg);
            }, 5);
          },
          // 默认报错信息会被 `cb()` 第二个参数覆盖
          message: 'Default error message'
        },
        required: [true, 'User phone number required']
      }
    });

    var User = db.model('User', userSchema);
    var user = new User();
    var error;

    user.phone = '555.0123';
    user.name = 'test';
    user.validate(function(error) {
      assert.ok(error);
      assert.equal(error.errors['phone'].message,
        '555.0123 is not a valid phone number!');
      assert.equal(error.errors['name'].message,
        'Validator failed for path `name` with value `test`');
    });
  
```

### 中间件middleware

中间件（pre和post钩子）是在异步函数执行时函数传入的控制函数，mongoose中所有的中间件都支持pre和post钩子。

#### pre钩子

pre钩子分为串行和并行两种，串行就是中间件一个接一个的执行，也就是上一个中间件调用next函数的时候，下一个执行。

```javascript
var schema = new Schema(..);
schema.pre('save', function(next) {
  // do stuff
  next();
});
```

并行中间件提供细粒度流控制

```javascript
var schema = new Schema(..);

// 只有第二个参数为‘true'时才表示并行执行
schema.pre('save', true, function(next, done) {
  // calling next kicks off the next middleware in parallel
  next();
  setTimeout(done, 100);
});
```



#### post中间件

post中间件在方法执行后调用

```javascript
schema.post('init', function(doc) {
  console.log('%s has been initialized from the db', doc._id);
});
schema.post('validate', function(doc) {
  console.log('%s has been validated (but not saved yet)', doc._id);
});
schema.post('save', function(doc) {
  console.log('%s has been saved', doc._id);
});
schema.post('remove', function(doc) {
  console.log('%s has been removed', doc._id);
});
```

如果在回调函数传入两个参数，mongoose会认为第二个参数是next函数，我们可以通过next触发下一个中间件

```javascript
// Takes 2 parameters: this is an asynchronous post hook
schema.post('save', function(doc, next) {
  setTimeout(function() {
    console.log('post1');
    // Kick off the second post hook
    next();
  }, 10);
});

// Will not execute until the first middleware calls `next()`
schema.post('save', function(doc, next) {
  console.log('post2');
  next();
});
```

### 连接Connections

我们可用mongoose.connect()连接，最简单的如```mongoose.connect('mongodb://localhost/myapp');```

,当然我们也可以在uri中指定多个参数```mongoose.connect('mongodb://username:password@host:port/database?options...');```

我们也可以用mongoose.createConnection()来连接，它返回一个新的连接,connection对象后面用于创建和检索models。

```javascript
const conn = mongoose.createConnection('mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]', options);
```



#### 操作缓存

意思就是我们不必等待连接建立成功就可以使用models,mongoose会先缓存model操作

```javascript
var MyModel = mongoose.model('Test', new Schema({ name: String }));
// 连接成功前操作会被挂起
MyModel.findOne(function(error, result) { /* ... */ });

setTimeout(function() {
  mongoose.connect('mongodb://localhost/myapp');
}, 60000);
```

如果要禁用缓存，可修改bufferCommands配置，也可以全局禁用bufferCommands

```javascript
mongoose.set('bufferCommands', false);
```

#### 选项

connect方法可以接受options参数,具体有哪些参数可以参考官方文档

```
mongoose.connect(uri,options)
```

```javascript
const options = {
  useMongoClient: true,
  autoIndex: false, // Don't build indexes
  reconnectTries: Number.MAX_VALUE, // Never stop trying to reconnect
  reconnectInterval: 500, // Reconnect every 500ms
  poolSize: 10, // Maintain up to 10 socket connections
  // If not connected, return errors immediately rather than waiting for reconnect
  bufferMaxEntries: 0
};
mongoose.connect(uri, options);
```

#### 回调

connect函数可以接受回调函数或者返回一个promise

```
mongoose.connect(uri,options,function(error){
})

mongoose.connect(uri,options).then(
 () => { /** ready to use. The `mongoose.connect()` promise resolves to undefined. */ },
  err => { /** handle initial connection error */ }
)
```

#### 注意

这里有个连接池的概念，无论是使用 `mongoose.connect` 或是 `mongoose.createConnection` 创建的连接， 都被纳入默认最大为 5 的连接池，可以通过 poolSize 选项调整：

```javascript
// With object options
mongoose.createConnection(uri, { poolSize: 4 });

const uri = 'mongodb://localhost/test?poolSize=4';
mongoose.createConnection(uri);
```

### 子文档Subdocument

子文档指的是嵌套在另一个文档中的文档，mongoose文档有两种不同的概念：子文档数组和单个嵌套子文档

```javascript
var childSchema = new Schema({ name: 'string' });

var parentSchema = new Schema({
  // Array of subdocuments
  children: [childSchema],
  // Single nested subdocuments. Caveat: single nested subdocs only work
  // in mongoose >= 4.2.0
  child: childSchema
});
```

子文档和普通文档类似，主要不同的是子文档不能单独保存，它们会在它们的顶级文档保存时保存

```javascript
var Parent = mongoose.model('Parent', parentSchema);
var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
parent.children[0].name = 'Matthew';

// `parent.children[0].save()` 无操作，虽然他触发了中间件
// 但是**没有**保存文档。你需要 save 他的父文档
parent.save(callback);
```



### 常用的操作：

1. **Model.find**
   Model.find(query,fields,options,callback)//fields和options都是可选参数
   简单查询
   Model.find({'csser.com':5},function(err,docs){docs是查询的结果数组})；
   只查询指定键的结果
   Model.find({},['first','last'],function(err,docs){//docs此时只包含文档的部分键值})

2. **Model.findOne**
   它只返回单个文档
   Model.findOne({age:5},function(err,doc){//doc是单个文档})；

3. **Model.findById**
   与findOne相同，但它接收文档的_id作为参数，返回单个文档。_id可以是字符串或者ObjectID对象
   Model.findById(obj._id,function(err,doc){//doc是单个文档})

4. **Model.count**
   返回符合条件的文档数
   Model.count(conditions,callback)

5. **Model.remove**
   删除符合条件的文档
   Model.remove(conditions,callback)

6. **Model.distinct**
   查询符合条件的文档并返回根据键分组的结果

   Model.distinct(field,conditions,callback)

7. **Model.where**
   当查询比较复杂的时候就用where
   Model.where('age').gte(25)

   where('tags').in(['movie','music','art'])
   .select('name','age','tags')
   .skip(20)
   .limit(10)
   .asc('age')
   .slaveOk()
   .hint({age:1,name:1})
   .run(callback)

8. **Model.$where**
   有时我们需要在MongoDB中使用JavaScript表达式进行查询，这时可以使用find({$where:javascript})方式，$where是一种快捷方式，并支持链式调用查询
   Model.$where('this.firstname===this.lastname').exec(callback)

9. **Model.update**

   使用update子句更新指定条件的文档，更新数据在发送到数据库服务器之前会改变模型的类型,注意update返回的是被修改的文档数量
   var conditions={name:'borne'}
   ,update={$inc:{visits:1}}
   ,options={multi:true}
   Model.update(conditions,update,options,callback)

   注意：为了向后兼容，所有顶级更新键如果不是原子操作命名的，会被统一按$set操作处理，例如：

   var queryb={name:'borne'};

   Model.update(query,{name:'jason borne'},options,callback)

   上面会被这样发送到数据库服务器的

   Model.update(query,{$set:{name:'jason borne'}},options,callback)

10. **查询API**
    如果不提供回调函数，所有这些方法都返回Query对象，它们都可以被再次修改，直到调用exec方法
    var query=Model.find({});

    query.where('field',5);

    query.limit(5);

    query.skip(100);

