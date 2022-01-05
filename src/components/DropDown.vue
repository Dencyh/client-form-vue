<template>
  <div class="dropdown-wrapper">
    <div class="input-simulation" @click.stop="handleClick">
      <button class="arrow-btn"></button>
    </div>
    <input
      class="input"
      placeholder="Выбрать..."
      type="text"
      :value="inputValue"
    />
    <ul v-show="visible" class="dropdown-datalist">
      <li
        v-for="group in groups"
        :key="group"
        :class="selectedOptionClass"
        @click="handleSelect"
      >
        {{ group }}
      </li>
    </ul>
    <!-- <div class="selected" v-if="selectedGroups">
      <div
        class="selected__option"
        v-for="group in selectedGroups"
        :key="group"
        @click="deleteSelected"
      >
        {{ group }}
      </div>
    </div> -->
  </div>
</template>
<script>
export default {
  name: "drop-down",
  props: {
    options: Array,
  },
  data() {
    return {
      groups: this.options,
      visible: false,
      selectedGroups: [],
      inputValue: "",
      selectedOptionClass: "dropdown-datalist__item",
    };
  },
  emits: ["updateGroups"],
  methods: {
    handleClick(e) {
      this.visible = !this.visible;
    },
    handleClickOutside(e) {
      if (e.target.tagName == "UL" || e.target.tagName == "LI") return;
      this.visible = false;
    },
    handleSelect(e) {
      if (!this.selectedGroups.includes(e.target.textContent)) {
        this.selectedGroups.push(e.target.textContent);
        e.target.className = "dropdown-datalist__item active";
      } else {
        const option = e.target.textContent;
        const index = this.selectedGroups.indexOf(option);
        this.selectedGroups.splice(index, 1);
        e.target.className = "dropdown-datalist__item";
      }

      this.inputValue = this.selectedGroups.join(", ");

      this.$emit("updateGroups", this.selectedGroups);
    },
    deleteSelected(e) {
      const option = e.target.textContent;
      const index = this.selectedGroups.indexOf(option);
      this.selectedGroups.splice(index, 1);
      this.inputValue = this.selectedGroups.join(", ");
    },
  },
  watch: {
    watchOption() {
      selectedGroups;
    },
  },
  mounted() {
    document.addEventListener("click", this.handleClickOutside.bind(this));
  },
  beforeDestroy() {
    document.removeEventListener("click", this.handleClickOutside);
  },
};
</script>
<style lang="scss" scoped>
@import "@/assets/variables";

.input {
  width: 100%;
}
.dropdown-datalist {
  padding: 10px;
  z-index: 1001;
  max-height: 200px;
  position: absolute;
  background-color: rgba(255, 255, 255, 0.776);
  border-radius: 10px;
  list-style: none;
  backdrop-filter: blur(6px);
  box-shadow: 0px 0px 6px rgba(0, 0, 0, 0.365);
  transform: translateY(8px);
  overflow: scroll;
  width: 100%;
  overflow-x: hidden;
}
.dropdown-wrapper {
  position: relative;
}

.input-simulation {
  width: 100%;
  height: 35px;
  position: absolute;
  cursor: pointer;
}
.arrow-btn {
  position: absolute;
  top: 10px;
  margin-left: 90%;
  cursor: pointer;
  padding: 50px;
  background: none;

  border: solid $darkGrey;
  border-width: 0 2px 2px 0;
  display: inline-block;
  padding: 2px;
  transform: rotate(45deg);
  -webkit-transform: rotate(45deg);
}

.dropdown-datalist.hidden {
  display: none;
}

::-webkit-scrollbar {
  width: 5px;
  background-color: none;
  border-radius: 25px;
}

::-webkit-scrollbar-track {
  border-radius: 25px;
}

::-webkit-scrollbar-thumb {
  border-radius: 25px;
  background: #ffffffb5;
  cursor: pointer;
}

.dropdown-datalist__item {
  margin: 8px 0 5px 5px;
  font-weight: 700;
  cursor: pointer;
  transition: all 100ms ease;
  border-radius: 15px;
  padding: 3px 10px;

  &:hover {
    color: $white;
    background-color: $blue;
  }
}
.dropdown-datalist__item.active {
  background-color: $blue;
  color: $white;
}

.selected {
  display: flex;
  flex-direction: row-reverse;
  justify-content: flex-end;
  align-self: start;
  position: absolute;
}

.selected__option {
  background: $blue;
  color: $white;
  padding: 5px 6px;
  margin: 10px 10px 0px 0px;
  font-weight: 700;
  border-radius: 15px;
  cursor: pointer;

  &:hover {
    background-color: $red;
  }
}
</style>

