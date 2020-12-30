MyPromise(ts)
```javascript
enum Status {
    Pending = 'Pending',
    Resolved = 'Resolved',
    Rejected = 'Rejected'
}

type Fn<U> = (...rest: any[]) => U;
class MyPromise<T> {
    status: Status = Status.Pending;
    value: any = null;
    resolveCallbacks: Fn<any>[] = [];
    rejectCallbacks: Fn<any>[] = [];
    
    static resolve(value?: any) {
        const curPromise: any = this instanceof MyPromise ? 
            this : new MyPromise(() => {});
        curPromise.value = value;
        setTimeout(function () {
            curPromise.resolveCallbacks.reduce(function(value, fn) {
                return fn.call(null, value);
            }, curPromise.value)
            curPromise.status = Status.Resolved;
        })
        return curPromise;
    }

    static reject(err?: any) {
        const curPromise: any = this instanceof MyPromise ? 
            this : new MyPromise(() => {});
        curPromise.value = err;
        setTimeout(function() {
            curPromise.rejectCallbacks.reduce(function(value, fn) {
                return fn.call(null, value);
            }, curPromise.value)
            curPromise.status = Status.Rejected;
        })
        return curPromise;
    }

    constructor(fn: (resolve: Fn<any>, reject: Fn<any>) => void) {
        try {
            fn(MyPromise.resolve.bind(this), MyPromise.reject.bind(this));
        }catch (err) {
            MyPromise.reject.bind(this, err);
        }
    }

    then(cb: Fn<T>) {
        this.status === Status.Resolved ? 
            cb(this.value) : 
            this.status === Status.Pending ?
                this.resolveCallbacks.push(cb) : null
    }

    catch(cb: Fn<T>) {
        this.status === Status.Rejected ? 
            cb(this.value) : 
                this.status === Status.Pending ?
                    this.rejectCallbacks.push(cb) : null
    }
}