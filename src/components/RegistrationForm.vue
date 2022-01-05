<template>
  <form class="form" @submit.prevent>
    <section class="section" v-if="currentStep == 1">
      <h2 class="section__header">Основные сведения</h2>
      <div class="section__wrapper">
        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('lastName')"
          >
            Введите фамилию
          </div>
          <labeled-input v-model="user.lastName" required
            >Фамилия</labeled-input
          >
        </div>

        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('firstName')"
          >
            Введите имя
          </div>
          <labeled-input v-model="user.firstName" required>Имя</labeled-input>
        </div>
        <labeled-input v-model="user.middleName">Отчество</labeled-input>

        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('birthDate')"
          >
            Укажите дату рождения
          </div>
          <input
            class="input"
            v-model="user.birthDate"
            type="date"
            :min="minDate"
            :max="today"
          />
          <label class="label"
            >Дата рождения<span class="required">*</span></label
          >
        </div>

        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('phone')"
          >
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
            v-if="
              user.clientGroups &&
              signupErrors &&
              signupErrors.includes('clientGroups')
            "
          >
            Выберите хотя бы одну группу клиентов
          </div>

          <drop-down
            @updateGroups="handleDropdown"
            :options="['VIP', 'Проблемные', 'ОМС']"
          ></drop-down>

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

    <section class="section" v-if="currentStep == 2">
      <h2 class="section__header">Адрес</h2>
      <div class="section__wrapper">
        <labeled-input type="number" v-model="user.address.zip"
          >Индекс</labeled-input
        >
        <labeled-input v-model="user.address.country">Страна</labeled-input>
        <labeled-input v-model="user.address.state">Область</labeled-input>

        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('city')"
          >
            Укажите город
          </div>
          <labeled-input v-model="user.address.city" required
            >Город</labeled-input
          >
        </div>
        <labeled-input v-model="user.address.street">Улица</labeled-input>
        <labeled-input v-model="user.address.building">Дом</labeled-input>
      </div>
    </section>

    <section class="section" v-if="currentStep == 3">
      <h2 class="section__header">Документы</h2>
      <div class="section__wrapper">
        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('type')"
          >
            Укажите документ
          </div>

          <select v-model="user.identityDocument.type">
            <option value="" disabled selected>Выбрать...</option>
            <option value="passport">Паспорт</option>
            <option value="birthCerificate">Свидетельство о рождении</option>
            <option value="driverLicense">Водительское удостоверение</option>
          </select>
          <label class="label" >Тип документа<span class="required">*</span></label>
        </div>
        <div class="input-container row">
        </div>

        <labeled-input v-model="user.identityDocument.series"
          >Серия</labeled-input
        >
        <labeled-input v-model="user.identityDocument.number"
          >Номер</labeled-input
        >
        <labeled-input v-model="user.identityDocument.authority"
          >Кем выдан</labeled-input
        >
        <div class="input-container">
          <div
            class="error"
            v-if="signupErrors && signupErrors.includes('dateOfIssue')"
          >
            Укажите дату выдачи документа
          </div>
          <input
            class="input"
            v-model="user.identityDocument.dateOfIssue"
            type="date"
            :min="minDate"
            :max="today"
          />
          <label class="label"
            >Дата выдачи<span class="required">*</span></label
          >
        </div>
      </div>
    </section>

    <div class="buttons">
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
    </div>
  </form>

  <div class="bg" ref="bgRef"></div>
  <div class="dialog" ref="dialogRef">
    <check-img />
    <h3>Пользователь зарегистрирован</h3>
  </div>
</template>
<script>
import useVuelidate from "@vuelidate/core";
import { minLength, required } from "@vuelidate/validators";
import DropDown from "@/components/DropDown.vue";
import CheckImg from "@/assets/CheckImg.vue";

const currentDay = `${new Intl.DateTimeFormat("ru-RU", "default").format(
  new Date(Date.now())
)}`
  .split(".")
  .reverse()
  .join("-");

export default {
  name: "registration-form",
  components: { DropDown, CheckImg },
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
        address: {
          city: { required },
        },
        identityDocument: {
          type: { required },
          dateOfIssue: { required },
        },
      },
    };
  },
  methods: {
    createUser() {
      this.v$.$validate();
      this.v$.$errors.forEach((error) => {
        this.signupErrors.push(error.$property);
      });
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

      /* Success message show */
      this.$refs.bgRef.classList.add("show");
      this.$refs.dialogRef.classList.add("show");

      setTimeout(() => {
        this.$refs.bgRef.classList.remove("show");
        this.$refs.dialogRef.classList.remove("show");
      }, 2500);
    },

    addSelected(e) {
      if (!this.user.clientGroups.includes(e.target.value)) {
        this.user.clientGroups.push(e.target.value);
      }
    },

    goNext() {
      this.v$.$validate();
      this.signupErrors = [];

      this.v$.$errors.forEach((error) => {
        this.signupErrors.push(error.$property);
      });

      if (this.currentStep == 1 && this.signupErrors.length < 4) {
        this.currentStep++;
        this.signupErrors = [];
        return;
      } else if (this.currentStep == 2 && this.signupErrors.length < 3) {
        this.signupErrors = [];
        this.currentStep++;
      }
    },

    goBack() {
      this.currentStep--;
    },
    handleDropdown(value) {
      this.user.clientGroups = value;
    },
  },
};
</script>

<style lang="scss">
@import "@/assets/variables";

.form {
  display: flex;
  flex-direction: column;
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

  .input-container {
    display: flex;
    flex-direction: column-reverse;
    justify-content: space-between;
    width: 100%;
    margin: 0;
    align-self: start;
    position: relative;
  }
  .input-container.row {
    flex-direction: row-reverse;
    justify-content: flex-end;
    align-self: start;
  }
  .dropdown {
    position: absolute;
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
    margin: 0px 2px 10px 25px;
    cursor: pointer;
  }
  .label.checkbox {
    margin-top: 0;
    margin-bottom: 10px;
  }
  .buttons {
    float: left;
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
.bg {
  display: none;
  position: absolute;
  min-width: 200vw;
  min-height: 200vh;
}
.bg.show {
  display: block;
  background: rgba(0, 0, 0, 0.201);
}
.dialog {
  margin: 20px 20px;
  padding: 30px;
  text-align: center;
  align-items: center;
  justify-content: spa;
  display: flex;
  flex-direction: column;
  top: -100px;
  opacity: 0;
  background-color: white;
  box-shadow: 0px 5px 10px rgba(0, 0, 0, 0.15);
  border-radius: 15px;
  z-index: 1002;
  position: absolute;

  transition: all 400ms ease;
}

.dialog.show {
  top: 40px;
  opacity: 1;
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