```javascript
function myNew(ctx, ...rest) {
    if(!ctx) { return; }
    const obj = {};
    obj.__proto__ = ctx.prototype;
    const result = ctx.call(obj, ...rest);
    return result instanceof Object ? result : obj;
}