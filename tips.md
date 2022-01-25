javascript regex

```js
// qq regex test
let re_qq = /[1-9][0-9]{4,}/;
let qq_str = "25555432";

let match = re_qq.test(qq_str) && qq_str.length < 12;
console.log(match)

// wx length
let re_phone = /^1(3\d|4[5-9]|5[0-35-9]|6[567]|7[0-8]|8\d|9[0-35-9])\d{8}$/;
let phone_str = "13388880000";

match = re_phone.test(phone_str);
console.log(match);
```
