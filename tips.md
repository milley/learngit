javascript regex

```js
// qq regex test
let re_qq = /[1-9][0-9]{4,}/;
let qq_str = "25555432";

let match = re_qq.test(qq_str) && qq_str.length < 12;
console.log(match)

// wx length
let re_phone = /^[1][3-8][0-9]{9}$/;
let phone_str = "13388880000";

match = re_phone.test(phone_str);
console.log(match);
```
