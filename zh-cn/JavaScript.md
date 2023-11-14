## 获取当前日期

> 获取当前日期，如果使用today.toISOString()方法，则日期少8个时区，加上8即可

```javascript
const today = new Date();
var hour = today.getHours() + 8;
today.setHours(hour);
let beginTime = util.formatDate("y-m-d", today.toISOString());
```

