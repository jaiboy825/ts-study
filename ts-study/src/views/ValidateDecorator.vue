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
  Password = "^(?=.*[A-Za-z])(?=.*d)(?=.*[@$!%*#?&])[A-Za-zd@$!%*#?&]{8,}$",
}

function validateDecorator() {
  return function (t: any, pk: string, d: PropertyDescriptor) {
    const method = d.value;
    d.value = function () {
      try {
        method();
      } catch (error) {
        console.log(error);
      }
    };
  };
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
      errMsg: "영어만 하셈",
    },
    {
      id: 2,
      type: validType.Korean,
      value: "",
      status: false,
      errMsg: "한글만 하셈",
    },
    {
      id: 3,
      type: validType.Number,
      value: "",
      status: false,
      errMsg: "숫자만 하셈",
    },
    {
      id: 4,
      type: validType.Email,
      value: "",
      status: false,
      errMsg: "이메일 형식 지키셈",
    },
    {
      id: 5,
      type: validType.Password,
      value: "",
      status: false,
      errMsg: "비밀번호형식 지키셈",
    },
  ];

  checkValidate = (item: validItem) => {
    switch (item.type) {
      case validType.English:
        Decorator.regex = regex.English;
        break;
      case validType.Korean:
        Decorator.regex = regex.Korean;
        break;
      case validType.Number:
        Decorator.regex = regex.Number;
        break;
      case validType.Email:
        Decorator.regex = regex.Email;
        break;
      case validType.Password:
        Decorator.regex = regex.Password;
        break;
      default:
        break;
    }
    Decorator.item = item;
    this.validate.map((x) => {
      if (x.id == item.id) {
        return this.validating();
      }
    });
  };

  @validateDecorator()
  validating() {
    const reg = new RegExp(Decorator.regex);
    Decorator.item.status = reg.test(Decorator.item.value);
    return Decorator.item;
  }
}
</script>

<style></style>
