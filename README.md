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
### 4.主要采用promise-async-async进行逻辑处理
#### 首先创建一个user_getCountryList函数，如下代码
```js
  const user_getCountryList = async function (req, res) {
    console.log("/v1/ref start -->" + JSON.stringify(req.body));
    try {
      const respondData = {
        status: res.statusCode,
        data: {},
        error: {}
      };
      const username = req.body.username;
      const userpwd = req.body.userpwd;
      const userage = req.body.userage;
      const usercityname = req.body.usercityname;
      const userstatename = req.body.userstatename;
      const usercountryname = req.body.usercountryname;
      const userInfoCountry = await findUserCountry({ name: usercountryname }, usercountryname);//查看国家
      const userInfoState = await findUserState({ name: userstatename }, userstatename);//查看州
      const userInfoCity = await findUserCity({ name: usercityname }, usercityname);//查看城市
      const userInfo = await findUser({ username: username, }, username,userpwd,userage);//查看用户信息
      const updateInfoUser = await updateUser({ _id: userInfo },userInfoCity);//更新用户信息
      const updateInfoCity = await updateCity({ _id: userInfoCity }, userInfoState);//更新城市信息
      const updateInfoState = await updateState({ _id: userInfoState }, userInfoCountry);//更新州信息
      return res.json(respondData);
    }
    catch (error) {
      //错误处理
      console.log("userCity error -->" + JSON.stringify(error));
      respondData.error = error;
      return res.json(respondData);
    }
  }
```
首先查看传入的国家在country中有没有，加入有，返回_id,没有就创建传入的国家名，并返回_id,查看findUserCountry函数对应的逻辑
```js
  const findUserCountry = async function (cnd, country) {
    console.log("findUserCountry start --> " + JSON.stringify(cnd));
    return new Promise(function (resolve, reject) {
      Country.findOne(cnd, function (error, data) {
        console.log("findUserCountry findOne  data --> " + JSON.stringify(data));
        if (error) {
          return reject(error);
        }
        if (data) {
          return resolve(data._id);
        } else {
          const userCountry = new Country({
            name: country
          });
          userCountry.save(function (err, data) {
            if (err) {
              console.log("userCountry.save err-->" + JSON.stringify(err));
              return reject(err);
            }
            console.log("userCountry-->" + JSON.stringify(data));
            return resolve(data._id);
          });
        }
      });
    })
  }
```
同理传入的州，城市，用户信息以同样的方式返回_id
#### 接下来就要进行关联user->city->state->country
通俗的说就是在User表中city保存City表中所需要的_id;也就是之前返回的_id这时就可以用到，可以参考updateUser函数
```js
   const updateUser = async function (cnd, cityid) {
    console.log("updateUser start --> " + JSON.stringify(cnd));
    return new Promise(function (resolve, reject) {
      User.update(cnd, { $set: { city: cityid } }, function (error, data) {
        console.log("updateUser findOne  data --> " + JSON.stringify(data));
        if (error) {
          return reject(error);
        }
        return resolve(data);
      });
    })
  }
```
这时就把City对应的_id写进了User表中，可以查看表，如图：
![User表中数据](https://github.com/wang-weifeng/picture/blob/master/mongoose-ref/city.png)
![City表中数据](https://github.com/wang-weifeng/picture/blob/master/mongoose-ref/user.png)
同理user->city->state->country数据都可以写进不同的表中。
### 5.使用populate关联查询

    
