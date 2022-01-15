# Daily-refined
## 2022.01.15
### vue 缓存函数
```js
const aysncCache = (fn) => {
  const cache = Object.create(null)
  return str => {
    const hit = cache[str]
    if(hit) {
      return hit
    } else {
      return cache[str] = fn()
    }
  }
}
```
### 并发请求 请求成功则剩下的函数返回相同的返回值，失败则返回相应的错误值
```js
const asyncCache = (generatorFn,symbol) = > {
  const cache = new Map()
  return (param) => {
    return new Promise((resolve, reject) => {
      const symbol = symbol || param
      const cacheConfig = cache.get(symbol)
      if (!cacheConfig) {
        cacheConfig = {
          res: null,
          exector: [{resolve, reject}]
        }
      } else {
        if (cacheConfig.res) {
          resolve(cacheConfig.res)
        }
        cacheConfig.exector.push({resolve, reject})
      }
      const {exector} = cacheConfig
      if (cacheConfig.exector.length === 1) {
        const next = async() => {
          try {
            if (!exector.length) return
            const res = await generatorFn()
            while(cacheConfig.exector.shift()) {
              exector.shift().resolve(res)
            }
          } catch (error) {
            const {reject} = exector.shift()
            reject(error)
            next()
          }
        }
        next()
      }
    })
  }
}
```
