<template>
  <div class="typeahead">
    <input
        class="typeahead-input"
        type="text"
        :placeholder="placeholder"
        v-model="query"
        @input="onInput"
        @blur="onBlur"
        @focus="onFocus"
        @keydown.down.prevent="onDown"
        @keydown.up.prevent="onUp"
        @keydown.enter.prevent="onEnter"
    />
    <div v-if="showSearchRes" class="res-list">
      <div
          class="res-list-item"
          :class="{
          'res-list-item-active': current == index,
        }"
          v-for="(item, index) in items"
          :key="index"
          :id="'list-item-' + index"
          @mousedown.prevent
          @click="selectItem(item)"
          @mouseenter="current = index"
      >
        <p
            class="res-list-item-text"
            :data-text="item.name"
            v-html="boldMatchText(item.name)"
        ></p>
      </div>
    </div>
  </div>

  <iframe
      v-if="selectedItem"
      class="gmap"
      style="border: 0"
      loading="lazy"
      allowfullscreen
      referrerpolicy="no-referrer-when-downgrade"
      :src="
      'https://www.google.com/maps/embed/v1/place?key=AIzaSyBox_43IiHuZiHi8Dg7fZC_EU8Ba8g7qro&q=' +
      mapQuery
    "
  >
  </iframe>
</template>

<script>
export default {
  props: {
    placeholder: String,
  },

  methods: {
    escapeRegExp(text) {
      return text.replace(/[-[\]{}()*+?.,\\^$|#\\s]/g, "\\$&");
    },
    boldMatchText(text) {
      const regexp = new RegExp(`(${this.escapeRegExp(this.query)})`, "ig");
      return text.replace(regexp, "<strong>$1</strong>");
    },
    selectItem(item) {
      this.selectedItem = item;
    },
    onInput() {
      this.focused = true;
      this.searchStates();
    },
    onBlur() {
      // this.focused = false;
    },
    onFocus() {
      this.focused = true;
      this.searchStates();
    },
    searchStates() {
      this.current = 0;
      const q = `
      {
        states(query: "${this.query}") {
          name
          abbreviation
        }
      }
      `;
      const ts = this;
      this.axios
          .post("http://localhost:4000/graphql", {
            query: q,
          })
          .then(function (response) {
            ts.items = response.data.data.states;
          })
          .catch(function (error) {
            console.log(error);
          });
    },
    onDown() {
      if (!this.focused || this.current >= this.items.length) {
        return;
      }
      this.current++;
      this.scrollTo();
    },
    onUp() {
      if (!this.focused || this.current <= 0) {
        return;
      }
      this.current--;
      this.scrollTo();
    },
    scrollTo() {
      setTimeout(() => {
        const active_node = document.querySelector(
            `#list-item-${this.current}`
        );
        const res_list_node = document.querySelector(`.res-list`);
        res_list_node.scrollTo(0, active_node.offsetTop);
      });
    },
    onEnter() {
      if (
          this.current &&
          this.current >= 0 &&
          this.current < this.items.length
      ) {
        this.selectItem(this.items[this.current]);
      }
    },
  },
  data() {
    return {
      query: "",
      items: [],
      current: 0,
      selectedItem: undefined,
      focused: false,
    };
  },
  computed: {
    showSearchRes() {
      const items = this.items;
      if (this.focused && items.length > 0) {
        return true;
      }

      return false;
    },
    mapQuery() {
      if (this.selectedItem !== undefined) {
        console.log(this.selectedItem);
        return this.selectedItem.name + ",+USA";
      }
      return "";
    },
  },
};
</script>

<style scoped>
.typeahead {
  width: 100%;
}

.typeahead-input {
  width: 100%;
  margin: auto;
  height: 50px;
  border: none;
  border-radius: 6px;
  padding: 20px;
}

.res-list {
  width: 25%;
  margin-top: 5px;
  border-radius: 5px;
  box-shadow: 1px 1px 1px white;
  background-color: whitesmoke;
  max-height: 200px;
  overflow-y: scroll;
  position: absolute;
  z-index: 1000;
}

.res-list-item {
  height: 40px;
  color: black;
  padding: 0px 20px;
}

.res-list-item-active {
  background-color: #2f7bfe;
  color: white;
}

.res-list-item-text {
  font-size: larger;
  margin: 5px;
}

.typeahead-input:focus {
  outline: none;
}

.gmap {
  width: 100%;
  height: 100%;
  margin: auto;
  margin-top: 20px;
}
</style>
