```javascript
Array.prototype.myReduce = function(cb, initValue) {
    if(!cb && typeof cb !== ) return this;
    let value = initValue || this[0];
    for(let i = !!initValue ? 0 : 1; i < this.length; i ++) {
        value = cb.call(null, value, this[i]);
    }
    return value;
}