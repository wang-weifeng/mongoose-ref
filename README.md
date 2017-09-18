# mongoose-ref
最近在做一个项目涉及到mongoose的关联查询等等，之前做的mysql，postgresql比较多，而mongoose用的都是比较简单的存储数据，简单查询等等。
刚开始涉及ref还是有点小晕的，查询了相关资源，也可以模模糊糊做出来，但是会有各种报错。痛下决心研究透彻点，晚上熬夜到2点终于有点小眉目了。
> * node v8.5.0
> * mongodb
> * 结合一个接口理解
> * 结合promise-async-await
> * 一般采用mvc模式，本文就直接在express里直接写了
### 1. 首先建立了一个mongoose-ref项目
直接使用了express -e mongoose-ref
### 2.在routes/index里连接mongodb数据库
```js
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/ref');
```
### 3.建立4个模型，用户：User,城市：City,省份：State,国家：Country
同时建立关联user->city->state->country
```js
  const Schema = mongoose.Schema;
  const ObjectId = Schema.Types.ObjectId;
  const UserSchema = new Schema({
    username: { type: String },
    userpwd: { type: String },
    userage: { type: Number },
    city: { type: Schema.Types.ObjectId, ref: 'City' },
  });
  const CitySchema = new Schema({
    name: { type: String },
    state: { type: Schema.Types.ObjectId, ref: 'State' }
  });
  const StateSchema = new Schema({
    name: { type: String },
    country: { type: Schema.Types.ObjectId, ref: 'Country' }
  });
  const CountrySchema = new Schema({
    name: { type: String }
  });
  const User = mongoose.model('User', UserSchema);
  const City = mongoose.model('City', CitySchema);
  const State = mongoose.model('State', StateSchema);
  const Country = mongoose.model('Country', CountrySchema);
```
    
