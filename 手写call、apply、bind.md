手写call

```javascript
Function.prototype.myCall = function(ctx = window, ...rest) {
  const fn = Symbol();
  ctx[fn] = this;
  const result = ctx[fn](...rest);
  delete ctx[fn];
  return result;
}
```

手写apply

```javascript
Function.prototype.myApply = function(ctx = window, ...rest) {
  const fn = Symbol();
  ctx[fn] = this;
  let result;
  if(!rest) {
    result = ctx[fn]();
  }else {
    result = ctx[fn](...rest)
  }
  delete ctx[fn];
  return result;
}
```

手写bind

```javascript
Function.prototype.myBind = function(ctx = window, ...rest) {
  const _this = this;
  return function F(...args) {
    if(this instanceof F) {
      return new _this(...rest, ...args);
    }else {
      return _this.call(ctx, ...rest, ...args)
    }
  }
}
```

