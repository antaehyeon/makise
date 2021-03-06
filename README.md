# makise
마키세는 이쁜 JSON 스키마 정의 언어입니다.

회사에서 사용하는 복잡한 JSON 포맷이 있는데 이것의 스펙을 좀 이쁘게 정의해놓고 싶어서 만든 사이드 프로젝트입니다.
만든이가 열심히 개밥먹기하고 있으니 믿고 사용하셔도 좋습니다(?)


## JSON Schema라는 게 있던데?
안 이쁘더라구요


## 설치하고 돌려보기

```sh
$ npm install -g makise
$ makise --help
```


## package.json 테스트 해보기

이 프로젝트 저장소는 node.js로 구현돼있어서, `package.json` 파일을 갖고 있는데요,
test 폴더에 들어있는 `package.json.makise` 스키마를 사용해서 `package.json` 파일이 정상적인지 체크할 수 있습니다.

```sh
$ makise test/package.json.makise package.json
```

아무 내용도 출력되지 않고 프로그램이 종료된다면 `package.json`이 정상적인 내용을 담고있다는 것을 뜻합니다.
`package.json`에 엉뚱한 타입의 데이터를 넣으면 무슨 일이 일어나는지도 한 번 확인해보세요.


## 브라우저에서 사용하기

```html
<script src="bundle.js"></script>
<script>
var schema = makise.schemaFromCode('this is number');
schema.validate(1); //=> true
schema.validate(true); //=> false
</script>
```


## 문법 훑어보기

### this 정의하기
마키세의 문법은 `this`를 정의하는 것에서부터 시작합니다.
```makise
this is number // 주석도 쓸 수 있어요
               /*>_<*/
```
위의 스키마는 숫자라고 평가되는 JSON들을 통과시킵니다.

### 복잡한 타입 정의하기
```makise
// 정수나 문자열중 아무거나
this is int or string

// 열거된 값들 중 하나의 값이 되기를 바랄 경우에는 다음과 같이 정의합니다.
day_of_week is ('월', '화', '수', '목', '금', '토', '일')

// 정수와 문자열이 번갈아가며 들어있는 배열을 표현하고 싶다면 다음과 같이 정의합니다.
int_string_pattern is [int, string, ...]

// JSON은 object의 모양일 때가 제일 많죠
something is {
    a: number, // a 필드가 숫자인지 확인
    b: number = 1, // b 필드가 숫자인지 확인하고,
                   // 없으면 1이라는 값을 갖고있는 것으로 간주
    c = 1, // b 필드와 똑같이 해석됩니다.
    d: string
}
// `something` 타입의 값에 `d` 속성의 값이 `'mail'`인 경우에는
something[d = 'mail'] is { // 다음 사항들을 더 체크합니다.
    e: *, // 뭐가 들어있건 말건... 정의가 안돼있어도 상관없어요.
    f: any, // 아무 값이나 들어있어도 됩니다.
            // 하지만 어떤 값이던지간에 꼭 채워주세요.
    *: void // 나머지 필드에는 아무 값도 정의되지 않았으면 좋겠네요
}
something[d = 'okabe'] throws '왠지 이 객체는 통과시켜주고 싶지 않네요'

// 기본타입에도 조건을 걸 수 있습니다.
int is number
int[not this % 1 = 0] throws '{{context}} is not int'
// {{context}}는 현재 타입 검사를 하는 대상의 표현식으로 치환됩니다.
```


## 기본 타입들
마키세에 미리 정의되어있는 타입은 다음과 같습니다.

* `*`: 정의되지 않은 경우를 포함해서 모든 값을 통과시켜주는 타입입니다.
* `any`: 값이 정의된 경우에 한해서 모든 값을 통과시켜주는 타입입니다.
* `void`: 정의되지 않은 경우를 제외하고 모든 값을 통과시켜주지 않는 타입입니다.
* `null`: `null` 값만 통과시키는 타입입니다.
* `number`: 굳이
* `string`: 설명을
* `boolean`: 안해도
* `object`: 아실거라
* `array`: 믿습니다


## 구현된 것들, 구현될 것들

* [x] `a is b`
* [x] `a throws "message"`
* [x] `a[expression] is b` conditional checker
    - [x] `a[field = value]`
    - [x] `a[field.sub_field]`
    - [x] `a[(b)]`
    - [x] `a[b < c]`, `a[b > c]`, `a[b <= c]`, `a[b >= c]`
    - [x] `a[b + c]`, `a[b - c]`
    - [x] `a[b * c]`, `a[b / c]`, `a[b % c]`
    - [x] `a[not b]`
    - [x] `a[b or c]`, `a[b and c]`
    - [x] `a[b?]`
* [x] `a is b or c`, `a is b and c`
* [ ] `a is not b` - 문법 디자인 고민 중
* [x] `a is (b, c)` enum checker
* [x] `a is (b, c,)` allow trailing comma
* [x] `a is {b: c}` object checker
    - [x] `a is {b: c = d}` default value
    - [x] `a is {b = c}` type inference
* [x] `a is [b, c]` array checker
    - [x] `a is [c, d, ...]` pattern checker
* [x] `a is b?` nullable type
* [x] `a is /(?:)/gi` regular expression matching
* [ ] validator code generation
