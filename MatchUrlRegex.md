# This is how you can match any URL using regex for Javascript
```js
let reg = /(https?:\/\/)(www\.)?([A-Za-z0-9]*)(\.[-a-zA-Z0-9@:%_\+.~#?&//=]*)/g
reg = new RegExp(reg);
let t = 'https://www.google.com';

if (t.match(reg)) {
  console.log("Successful match");
} else {
  console.log("No match");
}
```
