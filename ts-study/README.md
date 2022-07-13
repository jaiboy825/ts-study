# ts-study

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).

### 공부 내용

- Typescript 위주로 공부한 내용을 예제로 작성한 프로젝트임
- Vue3, Vuex, Typescript, router 적용되어있음

## 데코레이터

### 메서드 데코레이터

- 1. 받은 정보들을 토대로 메타데이터에서 가져옵니다.
- 2. 가져온 데이터가 있다면, 해당 아이템의 유형에 따라 정규식 변경
- 3. 정규식 test를 통해 성공 여부를 아이템의 status에 저장

```ts
function validateDecorator(
  target: any,
  propertyName: string,
  descriptor: TypedPropertyDescriptor<any>
) {
  let method = descriptor.value;
  descriptor.value = function () {
    let capitalParameters: number[] = Reflect.getOwnMetadata(
      checkCapitalMetadataKey,
      target,
      propertyName
    ); //metadata에서
    // CheckValidate Symbol metadata 키를 가지고,
    // 멤버에 대한 생성자 함수, 프로토타입 타겟에
    // checkValidate라는 property 키를 가진 애를 가져오기~
    if (capitalParameters) {
      // metadata에 저장이 되어있으면 ?
      for (let parameterIndex of capitalParameters) {
        //루프를 돌려~
        let regex_text = ""; // 정규식 문자 text
        switch (arguments[parameterIndex].type) {
          case validType.English:
            regex_text = regex.English;
            break;
          case validType.Korean:
            regex_text = regex.Korean;
            break;
          case validType.Number:
            regex_text = regex.Number;
            break;
          case validType.Email:
            regex_text = regex.Email;
            break;
          case validType.Password:
            regex_text = regex.Password;
            break;
          default:
            break;
        }
        const reg = new RegExp(regex_text); // 정규식 test용 객체 생성
        arguments[parameterIndex].status = reg.test(
          arguments[parameterIndex].value
        ); //정규식 성공 여부를 해당 아이템 status에 설정
      }
    }
    return method!.apply(this, arguments); //함수에 단일 배열 전달~
  };
}
```

### 파라미터 데코레이터

- 1. 파라미터에 연결함
- 2. existingCapitalParameters에
  - 메타데이터에 배열이 존재한다면 해당 배열
  - 없다면 빈 배열
- 3. 위에서 가져온 배열에 매개변수 인덱스 담아주기
- 4. metadata 정의

```ts
function parameterDecorator(
  target: any,
  propertyKey: string | symbol,
  parameterIndex: number
) {
  let existingCapitalParameters: number[] =
    Reflect.getOwnMetadata(checkCapitalMetadataKey, target, propertyKey) || [];
  existingCapitalParameters.push(parameterIndex); //metadata에서 가져온 배열에 해당 매개변수 인덱스를 담아주기
  Reflect.defineMetadata(
    checkCapitalMetadataKey,
    existingCapitalParameters,
    target,
    propertyKey
  );
  // 데코레이터 내부에 metadata 정의
}
```

### 적용 부분

```ts
  @validateDecorator
  checkValidate(@parameterDecorator item: validItem): void {
    console.log(item);
    // {
    //     "id": 4,
    //     "type": "Email",
    //     "value": "fdsafd@fdsafd.com",
    //     "status": true,
    //     "errMsg": "이메일 형식 (ex : email@email.com)"
    // }
  }
```
