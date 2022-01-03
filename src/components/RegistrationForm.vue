<template>
  <form class="form" @submit.prevent>
    <section class="section" v-if="currentStep == 1">
      <h2 class="section__header">Форма клиента</h2>
      <div class="section__wrapper">
        <div class="input-container">
          <div class="error" v-if="signupErrors.includes('lastName')">
            Введите фамилию
          </div>
          <labeled-input v-model="user.lastName" required
            >Фамилия</labeled-input
          >
        </div>

        <div class="input-container">
          <div class="error" v-if="signupErrors.includes('firstName')">
            Введите имя
          </div>
          <labeled-input v-model="user.firstName" required>Имя</labeled-input>
        </div>
        <labeled-input v-model="user.middleName">Отчество</labeled-input>

        <div class="input-container">
          <div class="error" v-if="signupErrors.includes('birthDate')">
            Укажите дату рождения
          </div>
          <input
            class="input"
            v-model="user.birthDate"
            required
            type="date"
            :min="minDate"
            :max="today"
          />
          <label class="label"
            >Дата рождения<span class="required">*</span></label
          >
        </div>

        <div class="input-container">
          <div class="error" v-if="signupErrors.includes('phone')">
            Номер должен состоять из 11 цифр
          </div>
          <phone-input
            formatRule="+7 (555) 555-55-55"
            v-model="user.phone"
            required
            >Телефон</phone-input
          >
        </div>

        <div class="input-container">
          <select v-model="user.sex">
            <option value="" disabled selected>Выбрать...</option>
            <option value="male">Мужской</option>
            <option value="female">Женский</option>
          </select>
          <label class="label">Пол</label>
        </div>

        <div class="input-container">
          <div
            class="error"
            v-if="user.clientGroups && signupErrors.includes('clientGroups')"
          >
            Выберите хотя бы одну группу клиентов
          </div>
          <div class="input-container row" v-if="user.clientGroups">
            <div
              class="label option"
              v-for="group in user.clientGroups"
              :key="group"
              @click="deleteOption"
            >
              {{ group }}
            </div>
          </div>
          <select v-model="user.group" @change="addSelected" class="input">
            <option value="" disabled selected>Выбрать...</option>
            <option value="VIP">VIP</option>
            <option value="Проблемные">Проблемные</option>
            <option value="ОМС">ОМС</option>
          </select>
          <label class="label">Группа клиентов</label>
        </div>

        <div class="input-container">
          <select v-model="user.attendingPhysician">
            <option value="" disabled selected>Выбрать...</option>
            <option value="Иванов">Иванов</option>
            <option value="Захаров">Захаров</option>
            <option value="Чернышева">Чернышева</option>
          </select>
          <label class="label">Лечащий врач</label>
        </div>
        <div class="input-container row">
          <input class="input checkbox" type="checkbox" v-model="user.smsOff" />
          <label class="label checkbox">Не отправлять СМС</label>
        </div>
      </div>
    </section>

    <section class="section" v-if="!this.v$.$error && currentStep == 2">
      <h2 class="section__header">Адрес</h2>
      <div class="section__wrapper">
        <labeled-input type="number" v-model="user.address.zip"
          >Индекс</labeled-input
        >
        <labeled-input v-model="user.address.country">Страна</labeled-input>
        <labeled-input v-model="user.address.state">Область</labeled-input>
        <labeled-input v-model="user.address.city" required
          >Город</labeled-input
        >
        <labeled-input v-model="user.address.street">Улица</labeled-input>
        <labeled-input v-model="user.address.building">Дом</labeled-input>
      </div>
    </section>

    <section class="section" v-if="!this.v$.$error && currentStep == 3">
      <h2 class="section__header">Документы</h2>
      <div class="section__wrapper">
        <labeled-input v-model="user.identityDocument.type" required
          >Тип документа</labeled-input
        >
        <labeled-input v-model="user.identityDocument.series"
          >Серия</labeled-input
        >
        <labeled-input v-model="user.identityDocument.number"
          >Номер</labeled-input
        >
        <labeled-input v-model="user.identityDocument.authority"
          >Кем выдан</labeled-input
        >
        <labeled-input v-model="user.identityDocument.dateOfIssue" required
          >Дата выдачи</labeled-input
        >
      </div>
    </section>

    <button
      class="btn"
      @click.prevent="goBack"
      v-if="currentStep > 1 && currentStep <= lastStep"
    >
      Назад
    </button>
    <button class="btn" @click.prevent="goNext" v-if="currentStep < lastStep">
      Далее
    </button>
    <button class="btn" @click="createUser" v-if="currentStep == lastStep">
      Подтвердить
    </button>
  </form>
</template>
<script>
import useVuelidate from "@vuelidate/core";
import { required, minLength } from "@vuelidate/validators";

const currentDay = `${new Intl.DateTimeFormat("ru-RU", "default").format(
  new Date(Date.now())
)}`
  .split(".")
  .reverse()
  .join("-");

export default {
  name: "registration-form",
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      user: {
        id: Number,
        lastName: "",
        firstName: "",
        middleName: "",
        birthDate: "",
        phone: "",
        sex: "",
        group: "",
        clientGroups: [],
        attendingPhysician: "",
        smsOff: false,
        address: {
          zip: "",
          country: "",
          state: "",
          city: "",
          street: "",
          building: "",
        },
        identityDocument: {
          type: "",
          series: "",
          number: "",
          authority: "",
          dateOfIssue: "",
        },
      },
      currentStep: 1,
      lastStep: 3,
      signupErrors: [],
      options: {
        year: "numeric",
        month: "numeric",
        day: "numeric",
      },
      today: currentDay,
      minDate: "1900-01-01",
    };
  },
  validations() {
    return {
      user: {
        firstName: { required },
        lastName: { required },
        birthDate: { required },
        phone: {
          minLengthValue: minLength(18),
          required,
        },
        clientGroups: { required },
      },
    };
  },
  methods: {
    createUser() {
      this.v$.$validate();
      if (this.v$.$error) return;
      const newUser = {
        id: Date.now(),
        lastName: this.user.lastName,
        firstName: this.user.firstName,
        middleName: this.user.middleName,
        birthDate: this.user.birthDate,
        phone: this.user.phone,
        sex: this.user.sex,
        clientGroups: JSON.parse(JSON.stringify(this.user.clientGroups)),
        attendingPhysician: this.user.attendingPhysician,
        smsOff: this.user.smsOff,
        address: {
          zip: this.user.address.zip,
          country: this.user.address.country,
          state: this.user.address.state,
          city: this.user.address.city,
          street: this.user.address,
          building: this.user.address.building,
        },
        identityDocument: {
          type: this.user.identityDocument.type,
          series: this.user.identityDocument.series,
          number: this.user.identityDocument.number,
          authority: this.user.identityDocument.authority,
          dateOfIssue: this.user.identityDocument.dateOfIssue,
        },
      };
      console.log("User created", newUser);
      this.currentStep = 1;

      /* Goes back to the first step and clears inputs */
      this.user = {
        id: Number,
        lastName: "",
        firstName: "",
        middleName: "",
        birthDate: "",
        phone: "",
        sex: "",
        group: "",
        clientGroups: [],
        attendingPhysician: "",
        smsOff: false,
        address: {
          zip: "",
          country: "",
          state: "",
          city: "",
          street: "",
          building: "",
        },
        identityDocument: {
          type: "",
          series: "",
          number: "",
          authority: "",
          dateOfIssue: "",
        },
      };
    },

    addSelected(e) {
      console.log(e.target.value);
      if (!this.user.clientGroups.includes(e.target.value)) {
        this.user.clientGroups.push(e.target.value);
      }
    },
    deleteOption(e) {
      const option = e.target.textContent;
      const index = this.user.clientGroups.indexOf(option);
      this.user.clientGroups.splice(index, 1);
    },
    goNext() {
      this.v$.$validate();
      if (!this.v$.$error) {
        this.currentStep++;
      }
      this.signupErrors = [];
      this.v$.$errors.forEach((error) => {
        this.signupErrors.push(error.$property);
      });
    },
    goBack() {
      this.currentStep--;
    },
  },
};
</script>

<style lang="scss">
@import "@/assets/variables";

.form {
  width: 50%;
  max-width: 800px;
  background: white;
  text-align: left;
  padding: 40px 60px;
  border-radius: 10px;
  box-shadow: 0px 5px 10px rgba(0, 0, 0, 0.15);

  .section {
    display: flex;
    flex-direction: column;
  }
  .section__wrapper {
    display: grid;
    grid-template-columns: 1fr 1fr;
    column-gap: 35px;
    row-gap: 30px;
  }

  .section__header {
    margin-bottom: 35px;
    color: $darkGrey;
    display: inline-block;
    font-size: 18px;
    font-weight: 700;
    letter-spacing: 1px;
  }
  .label {
    color: $darkGrey;
    display: inline-block;
    font-size: 14px;
    font-weight: 700;
    letter-spacing: 1px;
    align-self: flex-start;
  }

  .label.option {
    background: $blue;
    color: $white;
    padding: 5px 6px;
    margin: 10px 10px 0px 0px;

    border-radius: 15px;
    cursor: pointer;
  }

  .input-container {
    display: flex;
    flex-direction: column-reverse;
    justify-content: space-between;
    width: 100%;
    margin: 0;
    align-self: start;
  }
  .input-container.row {
    flex-direction: row-reverse;
    justify-content: flex-end;
    align-self: start;
  }

  .input,
  select {
    height: 35px;
    display: block;
    padding: 5px 6px;
    border: none;
    border-bottom: 1px solid #ddd;
    color: $black;
    outline: none;
    font-size: 16px;
    font-family: Avenir, Helvetica, Arial, sans-serif;
    color: $darkGrey;
    transition: all 100ms ease;

    &:focus + .label {
      color: $blue;
    }
    &:focus {
      border-color: $blue;
    }
  }
  .input.checkbox {
    width: 20px;
    height: 20px;
    margin: 10px 2px 0 15px;
  }
  .label.checkbox {
    margin-top: 10px;
  }

  .btn {
    margin: 30px 10px 0 0;
    background-color: $blue;
    padding: 10px;
    border: none;
    color: $white;
    border-radius: 25px;
    font-size: 16px;
    font-weight: 700;
    cursor: pointer;
    transition: all 200ms ease;

    &:hover {
      background-color: #087de3;
    }
  }
  .error {
    color: $red;
    font-size: 14px;
    margin-bottom: -15px;
    margin-top: 5px;
  }
}

@media (max-width: 1390px) {
  .form {
    width: 100%;
  }
}

@media (max-width: 800px) {
  .form {
    box-shadow: none;
    width: 100%;
    .section__wrapper {
      display: flex;
      flex-direction: column;
    }
  }
}

@media screen and (max-width: 500px), screen and (max-height: 400px) {
  .form {
    box-shadow: none;
    width: 100%;
    margin: 0;
    padding: 20px;
    .section__wrapper {
      display: flex;
      flex-direction: column;
    }
  }
}
</style>