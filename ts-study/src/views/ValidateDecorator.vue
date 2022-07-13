<template>
  <div class="container">
    <div class="validate-container col-6 mx-auto">
      <div class="card mb-2" v-for="item in validate" :key="item.id">
        <div class="card-body">
          <div class="input-container d-flex justify-content-between">
            <label :for="item.id">{{ item.type }}</label>
            <div class="form-group col-10">
              <input
                v-model="item.value"
                type="text"
                :id="item.id"
                class="col-12 mb-2 form-control"
                :class="
                  item.value == ''
                    ? ''
                    : item.status
                    ? 'is-valid'
                    : 'is-invalid'
                "
                required
                @input="checkValidate(item)"
              />
              <div class="invalid-feedback">
                {{ item.errMsg }}
              </div>
            </div>
          </div>
          <div class="input-status-text"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue } from "vue-class-component";
import "reflect-metadata";
enum validType {
  English = "English",
  Korean = "Korean",
  Number = "Number",
  Email = "Email",
  Password = "Password",
}

type validItem = {
  id: number;
  type: string;
  value: string;
  status: boolean;
  errMsg: string;
};

enum regex {
  English = "^[a-zA-Z]*$",
  Korean = "^[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]*$",
  Number = "^[0-9]*$",
  Email = "^([A-Z|a-z|0-9](.|_){0,1})+[A-Z|a-z|0-9]@([A-Z|a-z|0-9])+((.){0,1}[A-Z|a-z|0-9]){2}.[a-z]{2,3}$",
  // eslint-disable-next-line
  Password = "^(?=.*[A-Za-z])(?=.*d)(?=.*[@$!%*#?&])[A-Za-zd0-9@$!%*#?&]{8,}$",
}
const checkCapitalMetadataKey = Symbol("CheckValidate");

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

export default class Decorator extends Vue {
  static regex: string;
  static item: validItem;
  validate: validItem[] = [
    {
      id: 1,
      type: validType.English,
      value: "",
      status: false,
      errMsg: "영어만 입력하세요!",
    },
    {
      id: 2,
      type: validType.Korean,
      value: "",
      status: false,
      errMsg: "한글만 입력하세요!",
    },
    {
      id: 3,
      type: validType.Number,
      value: "",
      status: false,
      errMsg: "숫자만 입력하세요!!",
    },
    {
      id: 4,
      type: validType.Email,
      value: "",
      status: false,
      errMsg: "이메일 형식 (ex : email@email.com) 지켜주세요!!",
    },
    {
      id: 5,
      type: validType.Password,
      value: "",
      status: false,
      errMsg: "비밀번호 형식 (대문자 & 특수문자 1개 이상) 지켜주세요!!",
    },
  ];

  @validateDecorator
  checkValidate(@parameterDecorator item: validItem): void {
    console.log(item);
  }
}
</script>

<style></style>
