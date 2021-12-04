# tcp-exists
- A small zero-dependency library to check if some tcp endpoint (or many) exists. Can be used as a port scanner
- Scans `65536` endpoints in `~20sec` (via tcpExistsMany)
- Lib supports both: ESM and CJS 

## Install

```bash
npm i tcp-exists --save
```

## Description

### tcpExistsOne(host, port[, timeout[, signal]])
Arguments:
- `host` `<string>`
- `port` `<string> | <number>`
- `timeout` `<number>` - optional number of `ms`. **Default:** `100`
- `signal` `<AbortSignal>` - optional. An AbortSignal that may be used to close a socket and return result ASAP.

Returns:
- `<Promise<boolean>>`

### Usage
```javascript
import { tcpExistsOne } from 'tcp-exists'

const exist = await tcpExistsOne('8.8.8.8', 53, 25)
// check existance of endpoint 8.8.8.8:53 with timeout in 25ms

console.log(exist) // true
```



### tcpExistsChunk(endpoints[, options])
It is an async function to check multiple endpoints. If size of endpoints you want to check more than `4096` then recommended to use generator function `tcpExistsMany` or increase `timeout`.

#### Arguments:
- `endpoints` `<[string, string|number][]>` - array of `[host, port]`
- `options` `<object>` - optional
    - `timeout` `<number>` - optional number of `ms` to execute on chunk. **Default:** `1000`
    - `returnOnlyExisted` `<boolean>` - optional flag to exclude all non-existed results. **Default:** `true`
    - `signal` `<AbortSignal>` - optional. An AbortSignal that may be used to close a sockets and return result ASAP.

#### Returns:
- `<Promise<[string, string|number, boolean][]>>` - will return `array` of `[host, port, existed]`


#### Usage:
```javascript
import { tcpExistsChunk } from 'tcp-exists'

const endpoints = [
  ['8.8.8.8', 53],
  ['8.8.8.8', 80],
  ['8.8.8.8', 443],
  ['8.8.8.8', 8080]
]

const result = await tcpExistsChunk(endpoints)

console.log(result)
// all existed endpoints in format [host, port, existed][]
```




### tcpExistsMany(endpoints[, options])
It is an async generator, that's why the only way to use it with `for await (... of ...)`.

Useful to use with large amount of endpoints.

#### Arguments:
- `endpoints` `<[string, string|number][]>` - array of `[host, port]`
- `options` `<object>` - optional
    - `chunkSize` `<number>` - optional chunk size of endpoints to process at once. **Default:** `4096`
    - `timeout` `<number>` - optional number of `ms` to execute on chunk. **Default:** `1000`
    - `returnOnlyExisted` `<boolean>` - optional flag to exclude all non-existed results. **Default:** `true`
    - `signal` `<AbortSignal>` - optional. An AbortSignal that may be used to close a sockets, stop iteration and return last chunk result ASAP.

#### Returns:
- `<AsyncGenerator<[string, string|number, boolean][]>>` - generator will yield `array` of `[host, port, existed]`


#### Usage
```javascript
import { tcpExistsMany } from 'tcp-exists'

const host = '8.8.8.8'
const endpoints = Array.from({ length: 65535 }).map((_, port) => [host, port + 1]) // every port of 8.8.8.8

const result = []

for await (const existedEndpoints of tcpExistsMany(endpoints)) {
  result.push(...existedEndpoints) 
}

console.log(result)
// all existed endpoints in format [host, port, existed][]
```


## P.S.

There is better alternative of port scanner to use in shell written in rust [RustScan](https://github.com/RustScan/RustScan) (scans 65536 ports in **3s**)

License ([MIT](LICENSE))

