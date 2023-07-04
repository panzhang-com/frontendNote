### 1. Buffer api

- Buffer.alloc(size: number)
  分配一个大小为 size 的 Buffer

```javascript
const buf = Buffer.alloc(10);
const buf02 = Buffer.alloc(10, '1234567890');
const buf03 = Buffer.alloc(10, '1');
console.log(buf); // <Buffer 00 00 00 00 00 00 00 00 00 00>
log(buf02); // <Buffer 31 32 33 34 35 36 37 38 39 30>
log(buf03); // <Buffer 31 31 31 31 31 31 31 31 31 31>
```

- Buffer.byteLength(buf: Buffer)
  获取 Buffer 的字节长度
