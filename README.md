redis - a node.js redis client
===========================

[![Build Status](https://travis-ci.org/NodeRedis/node_redis.svg?branch=master)](https://travis-ci.org/NodeRedis/node_redis)
[![Coverage Status](https://coveralls.io/repos/NodeRedis/node_redis/badge.svg?branch=)](https://coveralls.io/r/NodeRedis/node_redis?branch=)
[![Windows Tests](https://img.shields.io/appveyor/ci/BridgeAR/node-redis/master.svg?label=Windows%20Tests)](https://ci.appveyor.com/project/BridgeAR/node-redis/branch/master)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/NodeRedis/node_redis?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

이것은 node.j를위한 완벽한 Redis 클라이언트입니다. 모든 Redis 명령을 지원하며 고성능에 중점을 둡니다.

함께 설치:

    npm install redis

## Usage Example

```js
var redis = require("redis"),
    client = redis.createClient();

// 데이터베이스 3을 선택하려면 0 대신 (default)
// client.select(3, function() { /* ... */ });

client.on("error", function (err) {
    console.log("Error " + err);
});

client.set("string key", "string val", redis.print);
client.hset("hash key", "hashtest 1", "some value", redis.print);
client.hset(["hash key", "hashtest 2", "some other value"], redis.print);
client.hkeys("hash key", function (err, replies) {
    console.log(replies.length + " replies:");
    replies.forEach(function (reply, i) {
        console.log("    " + i + ": " + reply);
    });
    client.quit();
});
```

그러면 다음과 같이 표시됩니다:

    mjr:~/work/node_redis (master)$ node example.js
    Reply: OK
    Reply: 0
    Reply: 0
    2 replies:
        0: hashtest 1
        1: hashtest 2
    mjr:~/work/node_redis (master)$

API는 완전히 비동기입니다. 서버에서 데이터를 다시 얻으려면 콜백을 사용해야합니다. API의 v.2.6부터 camelCase 및 snake_case 및 모든 옵션 / 변수 / 이벤트 등을 지원할 수 있습니다. camelCase는 Node.js 환경의 기본값이므로 사용하는 것이 좋습니다.

### Promises

#### Native Promises
If you are using node v8 or higher, util.promisify를 사용하여 [node_redis를 다음](https://nodejs.org/api/util.html#util_util_promisify_original) 과 같이 약속 할 수 있습니다 .
```js
const {promisify} = require('util');
const getAsync = promisify(client.get).bind(client);
```
지금 *getAsync은* 의 promisified 버전입니다 *client.get는* :
```js
// 'foo'값을 기대합니다 : 'bar'가 존재해야합니다 
// 그래서 client.get ( 'foo', cb); 다음과 같이 작성해야합니다 : 
return getAsync('foo').then(function(res) {
    console.log(res); // => 'bar'
});
```

or using [async await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function):
```js
async myFunc() {
    const res = await getAsync('foo');
    console.log(res);
}
```

#### Bluebird Promises
당신은 또한에 node_redis promisifying에 의해 약속 node_redis
[bluebird](https://github.com/petkaantonov/bluebird) 를 같이 사용할 수 있다:

```js
const redis = require('redis');
bluebird.promisifyAll(redis);
```

모든 node_redis 함수에 *비동기* 를 추가 합니다 (예 : return client.getAsync().then())

```js
// 'foo'값을 기대합니다 : 'bar'가 존재해야합니다 
// 그래서 client.get ( 'foo', cb); 다음과 같이 작성해야합니다. 
return client.getAsync('foo').then(function(res) {
    console.log(res); // => 'bar'
});

// prom와 함께 multi를 사용하면 다음과 같습니다 :

return client.multi().get('foo').execAsync().then(function(res) {
    console.log(res); // => 'bar'
});
```

### Sending Commands

각 Redis 명령은 `client`객체 에 대한 함수로 표시됩니다 . 모든 함수는 `args`배열과 선택적 `callback`Function 또는 가변 인수의 개별 인수와 선택적 콜백을 차례로 사용합니다. 
예 :

```js
client.hmset(["key", "test keys 1", "test val 1", "test keys 2", "test val 2"], function (err, res) {});
// 다음과 같이 작동합니다.
client.hmset("key", ["test keys 1", "test val 1", "test keys 2", "test val 2"], function (err, res) {});
// Or
client.hmset("key", "test keys 1", "test val 1", "test keys 2", "test val 2", function (err, res) {});
```

단일 인수가 실수로 여러 인수로 해석 될 수 있으므로 배열이 가능한 경우 (body-parser, 쿼리 문자열 또는 기타 메서드를 통해) 사용자 입력을 할 때는주의해야합니다.

두 형식 중 하나 `callback`는 선택 사항입니다:

```js
client.set("some key", "some val");
client.set(["some other key", "some val"]);
```

키가없는 경우 응답은 null입니다. [Redis Command Reference가](http://redis.io/commands) 다른 것을 언급 한 경우에만 null이되지 않습니다.

```js
client.get("missingkey", function(err, reply) {
    // reply is null when the key is missing
    console.log(reply);
});
```

Redis 명령 목록은 [Redis 명령 참조를 참조하십시오.](http://redis.io/commands)

응답에 대한 최소 구문 분석이 수행됩니다. 정수를 반환하는 명령은 JavaScript 숫자를 반환하고 배열은 JavaScript Array를 반환합니다. `HGETALL`해시 키로 키가 된 Object를 반환합니다. 모든 문자열은 설정에 따라 문자열 또는 버퍼로 반환됩니다. null, undefined 및 Boolean 값을 보내면 값이 문자열로 강제 변환됩니다.

# Redis Commands


이 라이브러리는 [Redis 명령에](https://redis.io/commands) 1 대 1로 매핑됩니다 . 캐시 라이브러리가 아니므로 자세한 사용법은 Redis 명령 페이지를 참조하십시오.

[SET 명령을](https://redis.io/commands/set) 사용하여 자동 만료 키 설정 
예:

```js
// this key will expire after 10 seconds
client.set('key', 'value!', 'EX', 10);
```

# API

## Connection and other Events

`client` 는 Redis 서버에 대한 연결 상태에 대한 일부 이벤트를 내 보냅니다.

### "ready"

`client``ready`연결이 설정되면 방출됩니다. 
`ready`이벤트 전에 발행 된 명령 은 대기열에 들어간 다음이 이벤트가 방출되기 바로 전에 재생됩니다.

### "connect"

`client``connect`스트림이 서버에 연결 되 자마자 방출 됩니다.

### "reconnecting"

`client``reconnecting`연결이 끊긴 후 Redis 서버에 다시 연결하려고 할 때 방출 됩니다. 
리스너에는 `delay` (이전 시도에서 ms로) 및 `attempt`(the attempt #) 속성이 포함 된 객체가 전달 됩니다.

### "error"

`client`발광한다 `error`레디 스 서버하거나 node_redis의 다른 발생 접속 오류가 발생할 때. 
콜백없이 명령을 사용하고 ReplyError가 발생하면 오류 리스너에 출력됩니다.

오류 수신기를 node_redis에 연결하십시오.

### "end"

`client``end`설정된 Redis 서버 연결이 닫히면 방출 됩니다.

### "drain" (deprecated)

`client``drain`Redis 서버에 대한 TCP 연결이 버퍼링되었지만 이제 쓰기가 가능할 때 방출 됩니다. 
이 이벤트는 Redis로 명령을 스트리밍하고 배압에 적응하는 데 사용할 수 있습니다.

스트림이 버퍼링 `client.should_buffer`되고 있는 경우는 true로 설정됩니다. 
그렇지 않으면 변수는 항상 false로 설정됩니다. 
그렇게하면 전송 속도를 줄이고받는 즉시 명령을 보내는 것을 재개 할시기를 결정할 수 있습니다 `drain`.

또한 각 명령의 반환 값을 확인할 수 있기 때문에 백 프레셔 표시기 (더 이상 사용되지 않음)가 반환됩니다. false가 돌려 주어 졌을 경우, 스트림은 버퍼링 할 필요가 있습니다.

### "warning"

`client``warning`암호가 설정되었지만 필요하지 않은 옵션과 더 이상 사용되지 않는 옵션 / 기능 / 유사 항목이 사용 되면 방출 됩니다.

### "idle" (deprecated)

`client``idle`응답을 기다리고있는 해결되지 않은 명령이 없을 때 방출 됩니다.

## redis.createClient()
`redis-server`노드와 동일한 시스템에서 실행중인 경우 , 포트 및 호스트의 기본값은 아마도 좋을 것이며 인수를 제공 할 필요가 없습니다. 
객체를 `createClient()`반환 `RedisClient`합니다. 
그렇지 않으면 `createClient()`다음 인수를 승인합니다:

* `redis.createClient([options])`
* `redis.createClient(unix_socket[, options])`
* `redis.createClient(redis_url[, options])`
* `redis.createClient(port[, host][, options])`

__Tip:__ Redis 서버가 클라이언트와 동일한 시스템에서 실행되는 경우 가능한 경우 유닉스 소켓 사용을 고려하여 처리량을 늘리십시오.

__Note:__ 사용 `'rediss://...`A의 프로토콜 `redis_url`에서 TLS 소켓 연결을 가능하게 할 것이다. 그러나 필요한 경우 추가 TLS 옵션을 전달해야 `options`합니다.

#### `options` object properties
| Property  | Default   | Description |
|-----------|-----------|-------------|
| host      | 127.0.0.1 | Redis 서버의 IP 주소 |
| port      | 6379      | Redis 서버의 포트 |
| path      | null      | Redis 서버의 UNIX 소켓 문자열 |
| url       | null      | Redis 서버의 URL입니다. 형식 : `[redis[s]:]//[[user][:password@]][host][:port][/db-number][?db=db-number[&password=bar[&option=value]]]`([IANA](http://www.iana.org/assignments/uri-schemes/prov/redis) 에서 자세한 정보 제공 ). |
| parser    | javascript | **Deprecated** 내장 JS 파서 [`javascript`](https://github.com/NodeRedis/node_redis/blob/master)또는 네이티브 [`hiredis`](https://github.com/NodeRedis/node_redis/blob/master)파서를 사용합니다. **참고** `node_redis` <2.6에서는 hiredis를 기본값으로 사용합니다. 이것은 v.2.6.0에서 변경되었습니다. |
| string_numbers | null | `true`로 설정, `node_redis`문자열 대신 자바 스크립트 숫자로 레디 스 번호 값을 반환합니다. 큰 숫자 (위 `Number.MAX_SAFE_INTEGER === 2^53`) 를 처리해야하는 경우 유용합니다 . Hiredis는이 동작을 할 수 없으므로이 옵션을 설정하면 옵션 `true`의 값에 관계없이 기본 javascript 파서가 사용됩니다 `parser`. |
| return_buffers | false | `true`로 설정하면 모든 응답이 문자열 대신 버퍼로 콜백으로 전송됩니다. |
| detect_buffers | false | `true`로 설정된 경우 응답은 버퍼로 콜백에 전송됩니다. 이 옵션을 사용하면 명령 단위로 버퍼와 문자열을 전환 할 수 있지만 `return_buffers`클라이언트의 모든 명령에 적용됩니다. **참고** : pubsub 모드에서는 제대로 작동하지 않습니다. 가입자는 항상 Strings 또는 Buffer를 반환해야합니다. |
| socket_keepalive | true | `true`로 설정하면 기본 소켓에서 연결 유지 기능이 활성화됩니다. |
| no_ready_check | false | Redis 서버에 대한 연결이 설정되면 서버가 여전히 디스크에서 데이터베이스를로드 중일 수 있습니다. 로드하는 동안 서버는 어떤 명령에도 응답하지 않습니다. 이 문제를 해결하려면 명령을 서버로 `node_redis`보내는 "준비 확인"기능이 `INFO`있어야합니다. `INFO`명령 의 응답 은 서버가 더 많은 명령을 사용할 준비가되었는지 여부를 나타냅니다. 준비가되면 이벤트를 내 `node_redis`보냅니다 `ready`. 을 (를) 설정`no_ready_check`하면 `true`이 확인이 금지됩니다. |
| enable_offline_queue |  true | 기본적으로 Redis 서버에 대한 활성 연결이없는 경우 명령은 대기열에 추가되고 연결이 설정되면 실행됩니다. 을 (를)로 설정 `enable_offline_queue`하면 `false`이 기능이 비활성화되고 콜백이 오류와 함께 즉시 실행되거나 콜백이 지정되지 않으면 오류가 발생합니다. |
| retry_max_delay | null | **Deprecated** *대신* **사용** *하십시오 retry_strategy.* 기본적으로 클라이언트가 연결을 시도하거나 실패 할 때마다 재 연결 지연이 거의 두 배가됩니다. 이 지연은 일반적으로 무한히 증가하지만 설정 `retry_max_delay`하면 밀리 초 단위로 제공되는 최대 값으로 제한됩니다. |
| connect_timeout | 3600000 | **Deprecated** *대신* **사용** *하십시오 retry_strategy.* 설정 `connect_timeout`은 클라이언트가 연결하고 다시 연결하는 총 시간을 제한합니다. 값은 밀리 초 단위로 제공되며 새 클라이언트가 생성되거나 연결이 끊어지는 순간부터 계산됩니다. 마지막 재시도는 시간 제한 시간에 정확히 발생합니다. 기본값은 기본 시스템 소켓 시간 초과가 초과 될 때까지 연결을 시도하고 1 시간이 경과 할 때까지 다시 연결을 시도하는 것입니다. |
| max_attempts | 0 | **Deprecated** *대신* **사용** *하십시오 retry_strategy.* 기본적으로 클라이언트는 연결될 때까지 다시 연결을 시도합니다. 설정 `max_attempts`은 연결 시도의 총량을 제한합니다. 이 값을 1로 설정하면 다시 연결을 시도 할 수 없습니다. |
| retry_unfulfilled_commands | false | `true`로 설정하면 연결이 끊긴 상태에서 완료되지 않은 모든 명령은 연결이 다시 설정된 후 다시 시도됩니다. 상태 변경 명령 (예 :)을 사용하는 경우이 값을 신중하게 사용하십시오 `incr`. 차단 명령을 사용하는 경우 특히 유용합니다. |
| password | null | 설정된 경우 클라이언트는 연결시 Redis auth 명령을 실행합니다. 별칭 `auth_pass`**참고** `node_redis` <2.5 사용해야합니다.`auth_pass` |
| db | null | 설정된 경우 클라이언트는 `select`연결시 Redis 명령을 실행 합니다. |
| family | IPv4 | 패밀리를 'IPv6'로 설정하면 IPv6을 강제로 사용할 수 있습니다. 패밀리 유형 사용 방법은 Node.js [net](https://nodejs.org/api/net.html) 또는 [dns](https://nodejs.org/api/dns.html) 모듈을 참조하십시오 . |
| disable_resubscribing | false | `true`으로 설정하면 연결 해제 후 클라이언트가 다시 가입하지 않습니다. |
| rename_commands | null | 원래 함수 대신 사용할 수 있도록 이름이 바뀐 명령을 사용하여 개체를 전달합니다. 예를 들어, KEYS 명령의 이름을 "DO NOT NOT USE"로 변경 한 경우 rename_commands 객체는 다음과 같습니다 `{ KEYS : "DO-NOT-USE" }`. 자세한 내용은 [Redis 보안 항목](http://redis.io/topics/security) 을 참조하십시오. |
| tls | null | [tls.connect](http://nodejs.org/api/tls.html#tls_tls_connect_port_host_options_callback) 로 전달하여 TLS 연결을 Redis에 설정 하는 옵션이 들어있는 객체입니다(예를 들어 터널을 통해 액세스 할 수 있도록 설정된 경우). |
| prefix | null | 사용 된 모든 키 앞에 접두사를 사용하는 문자열입니다 (예 :) `namespace:test`. 양해하여 주시기 바랍니다 `keys`명령은 접두사되지 않습니다. 이 `keys`명령은 "패턴"을 인수로 가지며 키가 없으며 접두어가 붙을 경우 Redis의 기존 키를 판별하는 것은 불가능합니다. |
| retry_strategy | function | 재시도 `attempt`, `total_retry_time`마지막으로 연결된 시간 이후 경과 한 시간`error`, 연결이 끊어진 이유 및 `times_connected`총 수를 나타내는 옵션 개체를 매개 변수로받는 함수입니다 . 이 함수에서 숫자를 반환하면 재시도가 밀리 초 단위로 정확하게 발생합니다. 숫자가 아닌 값을 반환하면 더 이상 다시 시도하지 않고 모든 오프라인 명령을 오류와 함께 삭제합니다. 특정 오류를 모든 오프라인 명령으로 리턴하려면 오류를 리턴하십시오. 아래 예. |


```js
var redis = require("redis");
var client = redis.createClient({detect_buffers: true});

client.set("foo_rand000000000000", "OK");

// This will return a JavaScript String
client.get("foo_rand000000000000", function (err, reply) {
    console.log(reply.toString()); // Will print `OK`
});

// This will return a Buffer since original key is specified as a Buffer
client.get(new Buffer("foo_rand000000000000"), function (err, reply) {
    console.log(reply.toString()); // Will print `<Buffer 4f 4b>`
});
client.quit();
```

retry_strategy example

```js
var client = redis.createClient({
    retry_strategy: function (options) {
        if (options.error && options.error.code === 'ECONNREFUSED') {
            // End reconnecting on a specific error and flush all commands with
            // a individual error
            return new Error('The server refused the connection');
        }
        if (options.total_retry_time > 1000 * 60 * 60) {
            // End reconnecting after a specific timeout and flush all commands
            // with a individual error
            return new Error('Retry time exhausted');
        }
        if (options.attempt > 10) {
            // End reconnecting with built in error
            return undefined;
        }
        // reconnect after
        return Math.min(options.attempt * 100, 3000);
    }
});
```

## client.auth(password[, callback])

인증이 필요한 Redis 서버에 연결할 때 연결 `AUTH` 후 명령을 첫 번째 명령으로 보내야합니다. 
재 연결, 준비 확인 등을 조정하는 것은 까다로울 수 있습니다.이를 쉽게하기 위해 재 연결을 포함하여 각 연결 후에이를 `client.auth()`숨기고 `password`보냅니다. `callback`첫 번째 `AUTH`명령에 대한 응답이 전송 된 후 한 번만 호출됩니다. 
참고 : 전화가 `client.auth()`준비 처리기 안에 있으면 안됩니다. 
이 작업을 잘못하면 `client`다음과 같은 오류가 발생합니다 `Error: Ready check failed: ERR operation not permitted`.

## backpressure

### stream

클라이언트는 사용 된 [stream](https://nodejs.org/api/stream.html) 을 노출 시켰고 `client.stream`스트림 또는 클라이언트가 명령 을 [buffer](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback) 경우 `client.should_buffer`. 
이 조합을 사용하면 명령을 보내기 전에 버퍼 상태를 확인하고 스트림 [drain](https://nodejs.org/api/stream.html#stream_event_drain) 이벤트를 수신하여 역압을 구현할 수 있습니다 .

## client.quit(callback)

이것은 종료 명령을 redis 서버로 보내고 실행중인 모든 명령이 제대로 처리 된 직후에 끝납니다. 
다시 연결하는 동안이 이름이 호출되면 (따라서 redis 서버에 대한 연결이없는 경우) 연결이 즉시 종료되고 대신 다시 연결됩니다. 
이 경우 모든 오프라인 명령은 오류와 함께 삭제됩니다.

## client.end(flush)

Redis 서버에 대한 연결을 강제로 닫으십시오. 모든 응답이 파싱 될 때까지 기다리지 않습니다. 깔끔하게 종료하려면 `client.quit()`위에서 언급 한대로 전화하십시오 .

다른 명령을 전혀 신경 쓰지 않는다면 flush를 true로 설정해야합니다. flush를 false로 설정하면 실행중인 모든 명령이 자동으로 실패합니다.

이 예에서는 응답을 읽기 전에 Redis 서버에 대한 연결을 닫습니다. 당신은 아마 이것을하고 싶지 않을 것입니다:

```js
var redis = require("redis"),
    client = redis.createClient();

client.set("foo_rand000000000000", "some fantastic value", function (err, reply) {
    // 이 (플러시 매개 변수를 true로 설정) 오류가 발생합니다 하나 
    // 또는 자동으로 실패하고이 콜백에서 호출되지 않습니다 모든 (false로 플러시 세트) 
    console.log(err);
});
client.end(true); // 더 이상의 명령은 처리되지 않습니다.
client.get("foo_rand000000000000", function (err, reply) {
    console.log(err); // => '연결이 이미 닫혔습니다.'
});
```

`client.end()` flush 매개 변수를 true로 설정하지 않으면 프로덕션 환경에 사용하지 않아야합니다!

## Error handling (>= v.2.6)

현재 다음과 같은 오류 하위 클래스가 있습니다.

- `RedisError`: 클라이언트 *가* 반환 한 *모든 오류*
- `ReplyError`하위 클래스 `RedisError`: **Redis** 자체 가 반환 한 모든 오류
- `AbortError`subclass of `RedisError`: 모든 이유로 인해 완료 할 수없는 모든 명령
- `ParserError`하위 클래스 `RedisError`: 파서 오류가 발생한 경우 반환됩니다 (발생하지 않아야 함).
- `AggregateError`subclass of `AbortError`: 콜백이없는 여러 개의 확인되지 않은 명령이 많은 `AbortError`s 대신 debug_mode에서 거부 된 경우에 발생합니다 .

모든 오류 클래스는 모듈에서 내 보냅니다.

예:
```js
var redis = require('./');
var assert = require('assert');
var client = redis.createClient();

client.on('error', function (err) {
    assert(err instanceof Error);
    assert(err instanceof redis.AbortError);
    assert(err instanceof redis.AggregateError);
    // 집합과 get은 여기에 집계됩니다.
    assert.strictEqual(err.errors.length, 2);
    assert.strictEqual(err.code, 'NR_CLOSED');
});
client.set('foo', 123, 'bar', function (err, res) { // Too many arguments
    assert(err instanceof redis.ReplyError); // => true
    assert.strictEqual(err.command, 'SET');
    assert.deepStrictEqual(err.args, ['foo', 123, 'bar']);

    redis.debug_mode = true;
    client.set('foo', 'bar');
    client.get('foo');
    process.nextTick(function () {
        // 명령이 아직 반환하지 않은 동안 강제로 연결을 닫습니다.
        client.end(true);
        redis.debug_mode = false;
    });
});

```

Every `ReplyError`에는 `command`대문자로 된 이름과 인수 ( `args`)가 들어 있습니다.

node_redis가 다른 오류로 인해 라이브러리 오류를 발생 시키면 트리거 오류가 반환 된 오류에 `origin`속성 으로 추가됩니다 .

**\*오류 코드***

node_redis는 `NR_CLOSED`클라이언트 연결이 끊어지면 오류 코드를 반환합니다 . 명령 미해결 명령이 거부 된 경우 `UNCERTAIN_STATE`코드가 반환됩니다. `CONNECTION_BROKEN`경우에 사용되는 에러 코드를 다시 연결 포기 node_redis.

## client.unref()

`unref()`Redis 서버에 대한 기본 소켓 연결을 호출 하여 보류중인 명령이 없으면 프로그램이 종료되도록합니다.

이 기능 은 **실험적** 기능이며 Redis 프로토콜의 하위 집합 만 지원합니다. 
클라이언트 상태가 Redis 서버에 저장된 모든 명령 (예 :`*SUBSCRIBE`차단 `BL*`명령)은 작동 *하지 않습니다*`.unref()`.

```js
var redis = require("redis");
var client = redis.createClient();

/*
    unref()를 호출하면이 프로그램이 get
    명령이 완료됩니다. 그렇지 않으면 클라이언트는
    client-server connection is alive.
*/
client.unref();
client.get("foo", function (err, value) {
    if (err) throw(err);
    console.log(value);
});
```

## Friendlier hash commands

대부분의 Redis 명령은 단일 문자열 또는 문자열 배열을 인수로 사용하며 응답은 단일 문자열 또는 문자열 배열로 다시 전송됩니다. 
해시 값을 처리 할 때 유용한 예외가 몇 가지 있습니다.

### client.hgetall(hash, callback)

HGETALL 명령의 응답은에 의해 JavaScript 오브젝트로 변환됩니다 `node_redis`. 
그렇게하면 JavaScript 구문을 사용하여 응답과 상호 작용할 수 있습니다.

예:

```js
client.hmset("hosts", "mjr", "1", "another", "23", "home", "1234");
client.hgetall("hosts", function (err, obj) {
    console.dir(obj);
});
```

Output:

```js
{ mjr: '1', another: '23', home: '1234' }
```

### client.hmset(hash, obj[, callback])

객체를 제공하여 해시의 여러 값을 설정할 수 있습니다.

```js
client.HMSET(key2, {
    "0123456789": "abcdefghij", // 참고 : 키와 값은 문자열로 강제 변환됩니다.
    "some manner of key": "a type of value"
});
```

이 객체의 속성과 값은 Redis 해시의 키와 값으로 설정됩니다.

### client.hmset(hash, key1, val1, ... keyn, valn, [callback])

목록을 제공하여 여러 값을 설정할 수도 있습니다.

```js
client.HMSET(key1, "0123456789", "abcdefghij", "some manner of key", "a type of value");
```

## Publish / Subscribe

Example of the publish / subscribe API. This program opens two
 이 프로그램은 두 개의 클라이언트 연결을 열어 그 중 하나에서 채널을 구독하고 다른 채널에서 해당 채널에 게시합니다:

```js
var redis = require("redis");
var sub = redis.createClient(), pub = redis.createClient();
var msg_count = 0;

sub.on("subscribe", function (channel, count) {
    pub.publish("a nice channel", "I am sending a message.");
    pub.publish("a nice channel", "I am sending a second message.");
    pub.publish("a nice channel", "I am sending my last message.");
});

sub.on("message", function (channel, message) {
    console.log("sub channel " + channel + ": " + message);
    msg_count += 1;
    if (msg_count === 3) {
        sub.unsubscribe();
        sub.quit();
        pub.quit();
    }
});

sub.subscribe("a nice channel");
```

클라이언트가 `SUBSCRIBE`또는을 발행하면 `PSUBSCRIBE`, 그 연결은 "구독자"모드가됩니다. 
이 시점에서 서브 스크립 션 세트를 수정하는 명령 만 유효하고 종료됩니다 (또한 redis 버전 ping에 따라 다름). 
서브 스크립 션 세트가 비어 있으면 연결이 일반 모드로 되돌아갑니다.

구독자 모드에서 Redis에 정기적 인 명령을 보내야하는 경우 새 클라이언트로 다른 연결을여십시오 (힌트 : 사용 `client.duplicate()`).

## Subscriber Events

클라이언트가 구독을 활성화 한 경우 다음 이벤트가 발생할 수 있습니다:

### "message" (channel, message)

클라이언트는 `message`활성 구독과 일치하는 수신 된 모든 메시지에 대해 방출 합니다. 
리스너에는 채널 이름 `channel`과 메시지 가 전달 됩니다 `message`.

### "pmessage" (pattern, channel, message)

클라이언트는 `pmessage`활성 구독 패턴과 일치하는 수신 된 모든 메시지에 대해 방출 합니다. 
리스너에는 `PSUBSCRIBE`as 와 함께 사용 된 원래 패턴 `pattern`, 보내는 채널 이름 as `channel`및 메시지가 전달됩니다 `message`.

### "message_buffer" (channel, message)

`message`예외가 있는 이벤트와 동일하며 항상 버퍼를 방출합니다. 
`message`the와 같은 시간에 이벤트 를 수신하면 `message_buffer`항상 문자열을 내 보냅니다.

### "pmessage_buffer" (pattern, channel, message)

`pmessage`예외가 있는 이벤트와 동일하며 항상 버퍼를 방출합니다. 
`pmessage`the와 같은 시간에 이벤트 를 수신하면 `pmessage_buffer`항상 문자열을 내 보냅니다.
### "subscribe" (channel, count)

클라이언트는 명령 `subscribe`에 대한 응답으로 방출 `SUBSCRIBE`합니다. 
리스너에는 `channel`이 클라이언트 의 채널 이름 과 새 구독 수가 전달 됩니다 `count`.

### "psubscribe" (pattern, count)

클라이언트는 명령 `psubscribe`에 대한 응답으로 방출 `PSUBSCRIBE`합니다. 
리스너에는 원래 패턴으로 전달되고 `pattern`,이 클라이언트에 대한 새로운 구독 수는 다음과 같습니다 `count`.

### "unsubscribe" (channel, count)

클라이언트는 명령 `unsubscribe`에 대한 응답으로 방출 `UNSUBSCRIBE`합니다. 
리스너에는 `channel`이 클라이언트 의 채널 이름 과 새 구독 수가 전달 됩니다 `count`. 때 `count`0,이 클라이언트는 가입자 모드를 떠나 더 이상 가입자 이벤트가 방출되지 않습니다.

### "punsubscribe" (pattern, count)

클라이언트는 명령 `punsubscribe`에 대한 응답으로 방출 `PUNSUBSCRIBE`합니다. 
리스너에는 `channel`이 클라이언트 의 채널 이름 과 새 구독 수가 전달 됩니다 `count`. 때 `count`0,이 클라이언트는 가입자 모드를 떠나 더 이상 가입자 이벤트가 방출되지 않습니다.

## client.multi([commands])

`MULTI`명령이 `EXEC`발행 될 때까지 명령이 대기 한 다음 모든 명령이 Redis에 의해 원자 적으로 실행됩니다. 인터페이스 in `node_redis`은 `Multi`호출 하여 개별 객체 를 반환하는 것 `client.multi()`입니다. 명령이 대기열에없는 경우 모든 명령이 롤백되고 아무 것도 실행되지 않습니다 (For
자세한 정보는
[transactions](http://redis.io/topics/transactions)).

```js
var redis  = require("./index"),
    client = redis.createClient(), set_size = 20;

client.sadd("bigset", "a member");
client.sadd("bigset", "another member");

while (set_size > 0) {
    client.sadd("bigset", "member " + set_size);
    set_size -= 1;
}

// 개별 콜백이있는 다중 체인
client.multi()
    .scard("bigset")
    .smembers("bigset")
    .keys("*", function (err, replies) {
    // 참고 :이 콜백의 코드는 원자가 아님
    // 이것은 .exec 호출이 끝난 후에 만 발생합니다.
        client.mget(replies, redis.print);
    })
    .dbsize()
    .exec(function (err, replies) {
        console.log("MULTI got " + replies.length + " replies");
        replies.forEach(function (reply, index) {
            console.log("Reply " + index + ": " + reply.toString());
        });
    });
```

### Multi.exec([callback])

`client.multi()``Multi`객체 를 반환하는 생성자입니다 . `Multi`객체는 객체와 동일한 명령 메소드를 모두 공유합니다 `client`. 명령 은 호출 `Multi`될 때까지 객체 내부에 대기합니다 `Multi.exec()`.

코드에 구문 오류가 있으면 EXECABORT 오류가 발생하고 모든 명령이 중단됩니다. 이 오류에는 `.errors` 구체적인 오류 가 포함 된 속성이 포함되어 있습니다. 모든 명령이 성공적으로 대기열에 남아 있고 오류가 결과 배열에 반환 될 명령을 처리하는 동안 오류가 발생합니다. onces가 실패한 것보다 다른 명령은 중단되지 않습니다.

`MULTI`위의 예 와 같이 명령을 연결하거나이 예제와 같이 일반 클라이언트 명령을 계속 보내면서 개별 명령을 대기열에 넣을 수 있습니다.

```js
var redis  = require("redis"),
    client = redis.createClient(), multi;

// 별도의 다중 명령 대기열을 시작합니다.
multi = client.multi();
multi.incr("incr thing", redis.print);
multi.incr("incr other thing", redis.print);

// 즉시 실행됩니다.
client.mset("incr thing", 100, "incr other thing", 1, redis.print);

// 다중 큐를 비우고 원자 적으로 실행합니다.
multi.exec(function (err, replies) {
    console.log(replies); // 101, 2
});
```

`MULTI`큐에 명령을 개별적 으로 추가하는 것 외에도 명령 및 인수 배열을 생성자에 전달할 수도 있습니다.

```js
var redis  = require("redis"),
    client = redis.createClient();

client.multi([
    ["mget", "multifoo", "multibar", redis.print],
    ["incr", "multifoo"],
    ["incr", "multibar"]
]).exec(function (err, replies) {
    console.log(replies);
});
```

### Multi.exec_atomic([callback])

Multi.exec과 동일하지만 단일 명령을 실행하면 트랜잭션이 사용되지 않는다는 차이점이 있습니다.

## client.batch([commands])

트랜잭션이없는 .multi와 동일합니다. 한 번에 많은 명령을 실행하지만 트랜잭션에 의존 할 필요가없는 경우에 권장됩니다.

`BATCH`명령이 `EXEC`발행 될 때까지 명령이 대기 한 다음 모든 명령이 Redis에 의해 원자 적으로 실행됩니다. 인터페이스 in `node_redis`은 `Batch`호출 하여 개별 객체 를 반환하는 것 `client.batch()`입니다. .batch와 .multi의 유일한 차이점은 트랜잭션이 사용되지 않는다는 것입니다. 결과는 다중 명령문과 마찬가지로 오류가 발생합니다. 그렇지 않으면 오류와 결과가 동시에 반환 될 수 있습니다.

한 번에 많은 명령을 실행하면 결과를 기다리지 않고 루프에서 동일한 명령을 실행하는 것과 비교하여 실행 속도를 현저히 높일 것입니다! 추가 비교를 위해 벤치 마크를 참조하십시오. 모든 명령은 해고 될 때까지 메모리에 보관된다는 것을 기억하십시오.

## Optimistic Locks

사용 `multi`당신이 당신의 수정 트랜잭션으로 실행되었는지 확인 할 수 있습니다,하지만 당신은 당신이 먼저 거기에 도착 확신 할 수 없다. 다른 클라이언트가 데이터로 작업하는 동안 키를 수정하면 어떻게 될까요?

이를 해결하기 위해 Redis는 MULTI와 함께 사용하기위한 [WATCH](https://redis.io/topics/transactions) 명령을 지원합니다 .

```js
var redis  = require("redis"),
    client = redis.createClient({ ... });

client.watch("foo", function( err ){
    if(err) throw err;

    client.get("foo", function(err, result) {
        if(err) throw err;
  
        // 결과 처리
        // 무겁고 시간이 많이 소요되는 작업은 여기에서

        client.multi()
            .set("foo", "some heavy computation")
            .exec(function(err, results) {
                
                /**
                 * err이 null 인 경우 Redis가 성공적으로 시도되었음을 의미합니다. 
                 * the operation.
                 */ 
                if(err) throw err;
                
                /**
                 * If results === null, 인 경우 동시 클라이언트
                 * 우리가 처리하는 동안 키를 변경 했으므로
                 * MULTI 명령의 실행이 수행되지 않았습니다.
                 * 
                 * 주의 사항 : MULTI의 실행 실패는 고려되지 않습니다.
                 * an error. So you will have err === null and results === null
                 */

            });
    });
});
```

위의 스 니펫은 `watch`with 의 올바른 사용법을 보여줍니다 `multi`. `multi`명령 실행 전에 감시 된 키가 변경 될 때마다 실행이 리턴 `null`됩니다. 정상적인 상황에서 실행은 연산 결과와 함께 값 배열을 반환합니다.

스 니펫에 설명 된대로 `multi`감시중인 명령 의 실행 실패 는 오류로 간주되지 않습니다. 예를 들어 클라이언트가 Redis에 연결할 수없는 경우 실행시 오류가 반환 될 수 있습니다.

`multi`명령 실행 실패를 볼 수있는 예제 는 다음과 같습니다.

```js
let clients = {};
clients.watcher = redis.createClient({ ... } );
clients.alterer = clients.watcher.duplicate();

clients.watcher.watch('foo',function(err) {
  if (err) { throw err; }
  // 다음 줄을 주석 처리하면 트랜잭션이 작동합니다.
  clients.alterer.set('foo',Math.random(), (err) => {if (err) { throw err; }})
  
  // 여기서 setTimeout을 사용하여 MULTI / EXEC가 SET 뒤에 올 것인지 확인합니다. 
  // 일반적으로, 당신은 순서를 보장하기 위해 콜백을 사용하지만, 나는 위의 SET 명령을 원하는 
  // 쉽게 아웃-수 언급 할 수 있습니다. 
  setTimeout(function() {
    clients.watcher
      .multi()
      .set('foo','abc')
      .set('bar','1234')
      .exec((err,results) => {
        if (err) { throw err; } 
        if (results === null) {
          console.log('transaction aborted because results were null');
        } else {
          console.log('transaction worked and returned',results)
        }
        clients.watcher.quit();
        clients.alterer.quit();
      });
  },1000);
});
```

### WATCH limitations

Redis WATCH는 *전체* 키 값 에서만 작동 합니다. 예를 들어, WATCH를 사용하면 수정을 위해 해시를 볼 수 있지만 해시의 특정 필드를 볼 수는 없습니다.

다음은 키를 볼 것입니다 `foo`및 `hello`아닌 필드 `hello` 해시를 `foo`:

```js
var redis  = require("redis"),
    client = redis.createClient({ ... });

client.hget( "foo", "hello", function(err, result){

    //이 필드의 값으로 일부 처리를 수행 한 후 처리합니다.
    
    client.watch("foo", "hello", function( err ){
        if(err) throw err;

         /**
         * WRONG: 이제 키 'foo'와 'hello'를보고 있습니다. 그렇지 않습니다.
         * 해시 'foo'의 'hello'필드를 보았습니다. 열쇠 'foo'
         * 해시를 참조하면이 명령은 이제 전체 해시를보고 있습니다.
         * 수정을 위해.  
         */ 
    });
} )

```

이 제한은 세트 (개별 세트 구성원을 볼 수 없음) 및 기타 콜렉션에도 적용됩니다.

## Monitor mode

Redis는 MONITOR다른 클라이언트 라이브러리 및 다른 컴퓨터를 포함한 모든 클라이언트 연결에서 Redis 서버가 수신 한 모든 명령을 볼 수 있는 명령을 지원 합니다.

monitor이벤트는 모니터링 클라이언트 자체를 포함하여 서버에 연결된 모든 클라이언트에서 해고 모든 명령에 방출 될 것입니다. 
monitor이벤트에 대한 콜백 은 명령 인수 배열과 원시 모니터링 문자열 인 Redis 서버의 타임 스탬프를 사용합니다.

예:

```js
var client  = require("redis").createClient();
client.monitor(function (err, res) {
    console.log("Entering monitoring mode.");
});
client.set('foo', 'bar');

client.on("monitor", function (time, args, raw_reply) {
    console.log(time + ": " + args); // 1458910076.446514:['set', 'foo', 'bar']
});
```

# Extras

당신이 알고 싶어하는 다른 것들.

## client.server_info

준비 프로브가 완료되면 INFO 명령의 결과가 `client.server_info`객체에 저장됩니다 .

`versions`키를 쉽게 비교 버전 문자열의 요소의 배열을 포함하고 있습니다.

    > client.server_info.redis_version
    '2.3.0'
    > client.server_info.versions
    [ 2, 3, 0 ]

## redis.print()

테스트 할 때 반환 값을 표시하는 편리한 콜백 함수. 예:

```js
var redis = require("redis"),
    client = redis.createClient();

client.on("connect", function () {
    client.set("foo_rand000000000000", "some fantastic value", redis.print);
    client.get("foo_rand000000000000", redis.print);
});
```

그러면 다음과 같이 인쇄됩니다:

    Reply: OK
    Reply: some fantastic value

이 프로그램은 클라이언트가 계속 연결되어 있기 때문에 정상적으로 종료되지 않습니다.

## Multi-word commands

redis multi-word 명령을 실행 `SCRIPT LOAD`하거나 `CLIENT LIST`두 번째 단어를 첫 번째 매개 변수로 전달 하려면 다음을 수행하십시오:

```js
client.script('load', 'return 1');
client.multi().script('load', 'return 1').exec(...);
client.multi([['script', 'load', 'return 1']]).exec(...);
```

## client.duplicate([options][, callback])

모든 현재 옵션을 복제하고 새 redisClient 인스턴스를 반환합니다. duplicate 함수에 전달 된 모든 옵션은 원래 옵션을 대체합니다. 콜백을 전달하면 클라이언트는 클라이언트가 준비 될 때까지 기다린 다음 콜백으로 반환합니다. 한편 오류가 발생하면 콜백 대신 오류가 반환됩니다.

duplicate ()를 사용하는 경우의 한 예는 연결 차단 redisation 명령 BRPOP, BLPOP 및 BRPOPLPUSH를 수용하는 것입니다. 이러한 명령이 비 차단 명령과 동일한 redisClient 인스턴스에서 사용되는 경우 차단되지 않는 명령이 차단 될 때까지 대기열에 보관 될 수 있습니다.

```js
var Redis=require('redis');
var client = Redis.createClient();
var clientBlocking = client.duplicate();

var get = function() {
    console.log("get called");
    client.get("any_key",function() { console.log("get returned"); });
    setTimeout( get, 1000 );
};
var brpop = function() {
    console.log("brpop called");
    clientBlocking.brpop("nonexistent", 5, function() {
        console.log("brpop return");
        setTimeout( brpop, 1000 );
    });
};
get();
brpop();
```

duplicate ()를 사용하는 또 다른 이유는 redis SELECT 명령을 통해 동일한 서버의 여러 DB에 액세스하는 경우입니다. 각 DB는 자체 연결을 사용할 수 있습니다.

## client.send_command(command_name[, [args][, callback]])


모든 Redis 명령이 `client`개체 에 추가되었습니다. 
그러나이 라이브러리가 업데이트되기 전에 새 명령이 도입되거나 개별 명령을 추가하려는 경우 `send_command()`Redis에 임의의 명령을 보낼 수 있습니다 .

모든 명령은 다중 대량 명령으로 전송됩니다. `args`인수의 배열이 될 수도 있고 생략되거나 / undefined가 될 수도 있습니다.

## redis.add_command(command_name)

add_command를 호출하면 프로토 타입에 새 명령이 추가됩니다. 
이 새로운 명령을 사용하여 호출 할 때 정확한 명령 이름이 사용됩니다. 다른 명령과 마찬가지로 임의의 인수를 사용할 수 있습니다.

## client.connected

Redis 서버에 대한 연결 상태를 추적하는 Boolean 입니다.

## client.command_queue_length

Redis 서버로 보내졌지만 아직 회신되지 않은 명령 수. 
 기능을 사용하여 연결되어있는 동안 명령의 최대 대기열 깊이를 강화할 수 있습니다.

## client.offline_queue_length

향후 연결을 위해 대기중인 명령 수입니다. 
이를 사용하여 연결 전 명령에 대해 최대 대기열 깊이를 적용 할 수 있습니다.

### Commands with Optional and Keyword arguments

이 옵션 사용 아무것도 적용 `[WITHSCORES]`또는 `[LIMIT offset count]`에 [redis.io/commands의](http://redis.io/commands) 문서를.

예:

```js
var args = [ 'myzset', 1, 'one', 2, 'two', 3, 'three', 99, 'ninety-nine' ];
client.zadd(args, function (err, response) {
    if (err) throw err;
    console.log('added '+response+' items.');

    // -Infinity and +Infinity also work
    var args1 = [ 'myzset', '+inf', '-inf' ];
    client.zrevrangebyscore(args1, function (err, response) {
        if (err) throw err;
        console.log('example1', response);
        // write your code here
    });

    var max = 3, min = 1, offset = 1, count = 2;
    var args2 = [ 'myzset', max, min, 'WITHSCORES', 'LIMIT', offset, count ];
    client.zrevrangebyscore(args2, function (err, response) {
        if (err) throw err;
        console.log('example2', response);
        // write your code here
    });
});
```

## Performance

`node_redis`일반적인 작업을 가능한 한 빨리 수행 하기 위해 많은 노력을 기울였습니다 .

```
Lenovo T450s, i7-5600U and 12gb memory
clients: 1, NodeJS: 6.2.0, Redis: 3.2.0, parser: javascript, connected by: tcp
         PING,         1/1 avg/max:   0.02/  5.26 2501ms total,   46916 ops/sec
         PING,  batch 50/1 avg/max:   0.06/  4.35 2501ms total,  755178 ops/sec
   SET 4B str,         1/1 avg/max:   0.02/  4.75 2501ms total,   40856 ops/sec
   SET 4B str,  batch 50/1 avg/max:   0.11/  1.51 2501ms total,  432727 ops/sec
   SET 4B buf,         1/1 avg/max:   0.05/  2.76 2501ms total,   20659 ops/sec
   SET 4B buf,  batch 50/1 avg/max:   0.25/  1.76 2501ms total,  194962 ops/sec
   GET 4B str,         1/1 avg/max:   0.02/  1.55 2501ms total,   45156 ops/sec
   GET 4B str,  batch 50/1 avg/max:   0.09/  3.15 2501ms total,  524110 ops/sec
   GET 4B buf,         1/1 avg/max:   0.02/  3.07 2501ms total,   44563 ops/sec
   GET 4B buf,  batch 50/1 avg/max:   0.10/  3.18 2501ms total,  473171 ops/sec
 SET 4KiB str,         1/1 avg/max:   0.03/  1.54 2501ms total,   32627 ops/sec
 SET 4KiB str,  batch 50/1 avg/max:   0.34/  1.89 2501ms total,  146861 ops/sec
 SET 4KiB buf,         1/1 avg/max:   0.05/  2.85 2501ms total,   20688 ops/sec
 SET 4KiB buf,  batch 50/1 avg/max:   0.36/  1.83 2501ms total,  138165 ops/sec
 GET 4KiB str,         1/1 avg/max:   0.02/  1.37 2501ms total,   39389 ops/sec
 GET 4KiB str,  batch 50/1 avg/max:   0.24/  1.81 2501ms total,  208157 ops/sec
 GET 4KiB buf,         1/1 avg/max:   0.02/  2.63 2501ms total,   39918 ops/sec
 GET 4KiB buf,  batch 50/1 avg/max:   0.31/  8.56 2501ms total,  161575 ops/sec
         INCR,         1/1 avg/max:   0.02/  4.69 2501ms total,   45685 ops/sec
         INCR,  batch 50/1 avg/max:   0.09/  3.06 2501ms total,  539964 ops/sec
        LPUSH,         1/1 avg/max:   0.02/  3.04 2501ms total,   41253 ops/sec
        LPUSH,  batch 50/1 avg/max:   0.12/  1.94 2501ms total,  425090 ops/sec
    LRANGE 10,         1/1 avg/max:   0.02/  2.28 2501ms total,   39850 ops/sec
    LRANGE 10,  batch 50/1 avg/max:   0.25/  1.85 2501ms total,  194302 ops/sec
   LRANGE 100,         1/1 avg/max:   0.05/  2.93 2501ms total,   21026 ops/sec
   LRANGE 100,  batch 50/1 avg/max:   1.52/  2.89 2501ms total,   32767 ops/sec
 SET 4MiB str,         1/1 avg/max:   5.16/ 15.55 2502ms total,     193 ops/sec
 SET 4MiB str,  batch 20/1 avg/max:  89.73/ 99.96 2513ms total,     223 ops/sec
 SET 4MiB buf,         1/1 avg/max:   2.23/  8.35 2501ms total,     446 ops/sec
 SET 4MiB buf,  batch 20/1 avg/max:  41.47/ 50.91 2530ms total,     482 ops/sec
 GET 4MiB str,         1/1 avg/max:   2.79/ 10.91 2502ms total,     358 ops/sec
 GET 4MiB str,  batch 20/1 avg/max: 101.61/118.11 2541ms total,     197 ops/sec
 GET 4MiB buf,         1/1 avg/max:   2.32/ 14.93 2502ms total,     430 ops/sec
 GET 4MiB buf,  batch 20/1 avg/max:  65.01/ 84.72 2536ms total,     308 ops/sec
 ```

## Debugging

디버그 출력을 얻으려면 `node_redis`응용 프로그램을 실행하십시오 `NODE_DEBUG=redis`.

또한 비동기 작업에 쓸모없는 것들에 반대되는 좋은 스택 트레이스가 생길 수 있습니다. 디버깅 출력이 아닌 스택 추적 만 좋으면 응용 프로그램을 개발 모드에서 대신 실행하십시오 ( `NODE_ENV=development`).

좋은 스택 추적은 개발 및 디버그 모드에서만 활성화되어 성능상의 불이익을 초래합니다.

**\*비교*** : 쓸모없는 스택 추적 :

___Comparison___:
Useless stack trace:
```
ReplyError: ERR wrong number of arguments for 'set' command
    at parseError (/home/ruben/repos/redis/node_modules/redis-parser/lib/parser.js:158:12)
    at parseType (/home/ruben/repos/redis/node_modules/redis-parser/lib/parser.js:219:14)
```
좋은 스택 추적 :
```
ReplyError: ERR wrong number of arguments for 'set' command
    at new Command (/home/ruben/repos/redis/lib/command.js:9:902)
    at RedisClient.set (/home/ruben/repos/redis/lib/commands.js:9:3238)
    at Context.<anonymous> (/home/ruben/repos/redis/test/good_stacks.spec.js:20:20)
    at callFnAsync (/home/ruben/repos/redis/node_modules/mocha/lib/runnable.js:349:8)
    at Test.Runnable.run (/home/ruben/repos/redis/node_modules/mocha/lib/runnable.js:301:7)
    at Runner.runTest (/home/ruben/repos/redis/node_modules/mocha/lib/runner.js:422:10)
    at /home/ruben/repos/redis/node_modules/mocha/lib/runner.js:528:12
    at next (/home/ruben/repos/redis/node_modules/mocha/lib/runner.js:342:14)
    at /home/ruben/repos/redis/node_modules/mocha/lib/runner.js:352:7
    at next (/home/ruben/repos/redis/node_modules/mocha/lib/runner.js:284:14)
    at Immediate._onImmediate (/home/ruben/repos/redis/node_modules/mocha/lib/runner.js:320:5)
    at processImmediate [as _immediateCallback] (timers.js:383:17)
```

## How to Contribute
- 끌어 오기 요청 또는 구현 / 변경하려는 항목에 대한 문제를 엽니 다. 우리는 어떤 도움도 기쁩니다!
- Google은 완전히 테스트 된 코드 만 수락한다는 점에 유의하십시오.

## Contributors

The original author of node_redis is [Matthew Ranney](https://github.com/mranney)

The current lead maintainer is [Ruben Bridgewater](https://github.com/BridgeAR)

Many [others](https://github.com/NodeRedis/node_redis/graphs/contributors)
contributed to `node_redis` too. Thanks to all of them!

## License

[MIT](LICENSE)

### Consolidation: It's time for celebration

지금은 주위에 두 개의 훌륭한 redis 클라이언트가 있으며 둘 다 서로 위에 몇 가지 장점이 있습니다. 우리는 ioredis와 node_redis에 대해 이야기합니다. 그래서 서로 협력하여 우리가 함께 일할 수있는 방법에 대해 이야기 한 후에 (즉, @luin과 @BridgeAR) 장기적으로 단일 라이브러리를 향해 작업하기로 결정했습니다. 그러나 단계적으로.

우선 우리는 라이브러리의 작은 부분을 다른 부분으로 나누어 동일한 코드를 사용할 수있게하려고합니다. 이러한 라이브러리는 NodeRedis 조직하에 유지 관리됩니다. 이것은 유지 관리 오버 헤드를 줄이고, 다른 사람들이 동일한 코드를 사용할 수있게하고, 필요하다면 다른 사람들이 두 라이브러리에 기여하는 것이 더 쉽습니다.

우리는 가능한 한 최고의 redis 경험을 제공하기 위해 함께 일하는이 단계에 매우 만족합니다.

도움을 받아 도움을 청하기 원하면 언제든지 저희에게 연락하십시오.

