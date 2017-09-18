# mongoose-ref
最近在做一个项目涉及到mongoose的关联查询等等，之前做的mysql，postgresql比较多，而mongoose用的都是比较简单的存储数据，简单查询等等。
刚开始涉及ref还是有点小晕的，查询了相关资源，也可以模模糊糊做出来，但是会有各种报错。痛下决心研究透彻写，晚上熬夜到2点终于成功。
```js
router.get('/', (req, res, next) => {
  res.render('index', { title: 'Express' });
});
```
    
