<template>
  <div class="searchbar">
    <input type="text" placeholder="想吃啥？按下搜尋幫您推薦！" v-model="query.locName" />
    <button @click="getUrl()">
      <span class="material-symbols-outlined">search</span>
    </button>
  </div>

  <div class="search-options">
    <div class="options">
      <button
        @click="selectOption('price')"
        :class="{
          active: choosedOption == 'price',
          selected: query.price != null,
        }"
      >
        <span v-if="query.price == null">價錢</span>
        <span v-else><span v-for="i in query.price" :key="i">$</span></span>
      </button>

      <button
        @click="selectOption('distance')"
        :class="{
          active: choosedOption == 'distance',
          selected: query.distance != 0,
        }"
      >
        <span v-if="query.distance == 0">距離</span>
        <span v-else>{{ query.distance }}m</span>
      </button>

      <button
        @click="selectOption('tagName')"
        :class="{
          active: choosedOption == 'tagName',
          selected: query.tagName != null,
        }"
      >
        <span v-if="query.tagName == null">類別</span>
        <span v-else id="tagName">{{ query.tagName }}</span>
      </button>

      <button
        @click="query.opening = !query.opening"
        :class="{ selected: query.opening }"
      >
        營業中
      </button>
    </div>

    <div class="semi-options">
      <div class="price" v-if="choosedOption == 'price'">
        <button @click="selectPrice('$')">$</button>
        <button @click="selectPrice('$$')">$$</button>
        <button @click="selectPrice('$$$')">$$$</button>
      </div>

      <div class="distance" v-if="choosedOption == 'distance'">
        <input
          type="range"
          min="0"
          max="2000"
          step="50"
          v-model="query.distance"
        />
        <span>{{ query.distance }}m</span>
      </div>

      <div class="tagName" v-if="choosedOption == 'tagName'">
        <button :key="i" v-for="(tag, i) in tags" @click="selectType(tag.name)">{{tag.name}}</button>
      </div>
    </div>
  </div>
</template>

<script>
import { getApi, popup } from "@/GlobalSettings";

export default {
  name: "NeSearchbar",
  props: {},
  data() {
    return {
      query: {
        locName: null,
        price: null,
        distance: 0,
        tagName: null,
        opening: false,
      },
      choosedOption: null,
      parameters: "",
      tags: null,
      isDone: false,
    };
  },

  created() {
    this.getTags()
  },

  setup() {
    const api = getApi();
    return {
      api,
      popup,
    };
  },

  methods: {
    selectOption(option) {
      if (this.choosedOption != option) {
        this.choosedOption = option;
        if (option == "distance") {
          this.query[option] = 0;
        } else {
          this.query[option] = null;
        }
      } else {
        this.choosedOption = null;
      }
    },

    selectPrice(price) {
      this.query.price = price;
      this.choosedOption = null;
    },

    selectType(tagName) {
      this.query.tagName = tagName;
      this.choosedOption = null;
    },

    getUrl() {
      const _this = this;

      if (this.query.locName) {
        this.parameters += "&locName=" + this.query.locName;
      }

      if (this.query.price !== null) {
        this.parameters += "&priceLevels=" + this.query.price.length + (this.query.price.length + 1);
      }

      if (this.query.distance) {
        this.parameters += "&distance=" + this.query.distance;
      } 

      if (this.query.tagName) {
        this.parameters += "&tagNames=" + this.query.tagName;
      }

      if (this.query.opening) {
        this.parameters += "&opening=" + this.query.opening;
      }

      if (this.parameters == ""){
        this.parameters += "&"
      }

      this.isDone = true;
    },

    getTags() {
      const _this = this
      _this.axios
        .get(this.api + "tag/list/?page=0&eachPageNum=8&drop=NOT_USED&orderby=FREQ_ASC", {
          withCredentials: true,
        })
        .then((res) => {
          if (res.data.state === "success") {
            this.tags = res.data.data.tags
          } else {
            const errorMsg = res.data;
            console.log(errorMsg);
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    }
  },

  watch: {
    isDone: function(){
      this.$router.push({
          path: `/searchresult/${this.parameters}`
      })
    }
  }
};
</script>

<style scoped lang="scss">
.searchbar {
  align-self: center;
  position: relative;
  margin-top: 36px;
  padding: 0px 18px;
  input {
    border-radius: 20px;
    border: 1px solid #dbd5d5;
    padding-left: 24px;
    width: 100%;
    height: 40px;
    box-shadow: inset 0px 1px 2px #00000029;
    font-size: 14px;
    &::placeholder {
      line-height: 20px;
      letter-spacing: 0px;
      color: #707070;
    }
  }
  button {
    border: none;
    background-color: transparent;
    position: absolute;
    height: 40px;
    width: 40px;
    right: 30px;
    top: 0px;
    span {
      color: #5f5c58;
      opacity: 0.78;
      line-height: 38px;
    }
    & :hover {
      cursor: pointer;
    }
  }
}

.search-options {
  padding: 0px 18px;
  margin-top: 14px;
  .options {
    display: flex;
    margin: 0 -5px;
    button {
      flex: 1;
      margin: 0px 5px;
      height: 28px;
      border: 1px solid #dbd5d5;
      text-align: center;
      font-weight: 600;
      font-size: 14px;
      line-height: 20px;
      color: #777777;
      background-color: #ffffff;
      border-radius: 20px;
      position: relative;
      white-space: nowrap;
      #tagName {
        display: inline-block;
        max-width: calc((100vw - 66px) / 4 - 20px);
        overflow: scroll;
        margin: 2.75px 0px -1.5px 0px;
        font-size: 14px;
      }
      &.active {
        background-color: #efefef;
      }
      &.active::after {
        content: "";
        display: block;
        position: absolute;
        width: 20px;
        height: 20px;
        background: #efefef;
        transform: translateX(-50%) rotate(45deg);
        top: 30.86px;
        left: 50%;
        z-index: -1;
      }
      &.selected {
        background-color: #f0e4d4;
      }
    }
  }

  .semi-options {
    .price {
      margin-top: 12px;
      padding: 6px 2px;
      border-radius: 20px;
      background-color: #efefef;
      display: inline-block;
      button {
        width: 60px;
        height: 28px;
        border: 1px solid #dbd5d5;
        text-align: center;
        font-weight: 600;
        font-size: 14px;
        line-height: 20px;
        color: #777777;
        background-color: #ffffff;
        border-radius: 20px;
        margin-left: 5px;
        margin-right: 5px;
      }
    }

    .distance {
      margin-top: 12px;
      padding: 5px 12px;
      border-radius: 20px;
      background-color: #efefef;
      display: inline-flex;
      input {
        -webkit-appearance: none;
        background-color: #efefef;
        &::-webkit-slider-runnable-track {
          -webkit-appearance: none;
          background-color: #707070;
          height: 4px;
        }
        &::-webkit-slider-thumb {
          -webkit-appearance: none;
          width: 16px;
          height: 16px;
          border-radius: 50%;
          background-color: #efefef;
          border: 1px solid #999;
          margin-top: -5.5px;
        }
      }
      span {
        background: #ffffff;
        border-radius: 20px;
        color: #777777;
        font-weight: 600;
        font-size: 14px;
        line-height: 30px;
        margin-left: 25px;
        display: inline-block;
        text-align: center;
        width: 70px;
      }
    }

    .tagName {
      margin-top: 12px;
      padding: 3px 6px;
      border-radius: 20px;
      background-color: #efefef;
      overflow: scroll;
      white-space: nowrap;
      &::-webkit-scrollbar {
        display: none;
      }
      button {
        border: 1px solid #dbd5d5;
        text-align: center;
        font-weight: 600;
        font-size: 14px;
        line-height: 28px;
        color: #777777;
        background-color: #ffffff;
        border-radius: 20px;
        padding: 0px 16px;
      }
    }
  }
}
</style>
