<template>
  <v-ons-page>
    <v-ons-toolbar>
      <div class="left">
        <div class="toolbar-heading">検索</div>
      </div>
    </v-ons-toolbar>
    <v-ons-list>
      <div v-for="item in users" :key="item.user_id">
        <v-ons-list-item modifier="nodivider">
          <div class="left image-list">
            <img class="list-item__thumbnail" :src="`http://a1333.ml/img/profile/${item.uuid}.jpg`" @click="showProfileImageModalWindow(item.user_id)">
          </div>
          <div class="center" @click="showProfileDetailModalWindow(item.user_id)">
            <span class="list-item__title">{{ item.name }}</span>
            <span class="list-item__title color-gray">{{ `${getGenderString(item.gender)} / ${item.age}歳 / ${item.prefecture}` }}</span>
          </div>
        </v-ons-list-item>
        <v-ons-list-item modifier="nodivider">
          <div class="center" @click="showProfileDetailModalWindow(item.user_id)">
            {{item.comment}}
          </div>
        </v-ons-list-item>
        <v-ons-list-item modifier="longdivider" class="longdivider gray-btns">
          <div class="left" v-if="item.user_id !== $root.userId">
            <ons-button  @click="sendHeart(item.user_id)" > <v-ons-icon icon="fa-heart" class="color-pink pl10" /> いいね</ons-button>
          </div>
          <div class="center">
            <ons-button  v-if="$root.userId != item.user_id" @click="showMessagePage(item.user_id)"><v-ons-icon icon="fa-envelope"></v-ons-icon> トーク</ons-button>
          </div>
          <div class="right">
            <ons-button class="triangle-btn" v-if="item.user_id !== $root.userId" @click="sendReport(item.user_id, item.id)">
              <v-ons-icon icon="fa-exclamation-triangle" class="color-gray"  /> 通報
            </ons-button>
          </div>
        </v-ons-list-item>
      </div>
    </v-ons-list>

    <v-ons-fab ripple position="bottom right" class="bottom-pencil">
      <v-ons-icon icon="fa-search" @click="searchModalWindow" />
    </v-ons-fab>

    <v-ons-modal v-if="searchModalWindowVisible === true" :visible="searchModalWindowVisible">
      <v-ons-list class="modal-list">
        <v-ons-list-item>
          <v-ons-row>
            <v-ons-col class="label">
              性別
            </v-ons-col>
            <v-ons-col width="224px">
              <div class="f-left">
                <v-ons-button class="left-btn" :class="gender == -1 ? 'active' : ''" @click="gender = -1">指定なし</v-ons-button>
              </div>
              <div class="f-left">
                <v-ons-button class="no-radius"  :class="gender == 0 ? 'active' : ''"  @click="gender = 0">男性</v-ons-button>
              </div>
              <div class="f-left">
                <v-ons-button class="right-btn"  :class="gender == 1 ? 'active' : ''"  @click="gender = 1">女性</v-ons-button>
              </div>
            </v-ons-col>
          </v-ons-row>
        </v-ons-list-item>
        <v-ons-list-item>
          <v-ons-row>
            <v-ons-col class="label">
              年齢
            </v-ons-col>
            <v-ons-col width="130px">
              {{ageValue[0]}} - {{ageValue[1]}}
            </v-ons-col>
          </v-ons-row>
          <v-ons-row>
            <v-ons-col class="right pt-5">
              <vue-slider v-model="ageValue" v-bind="options" ref="ageSelector"></vue-slider>
            </v-ons-col>
          </v-ons-row>
        </v-ons-list-item>
        <v-ons-list-item>
          <v-ons-row>
            <v-ons-col class="label">
              都道府県
            </v-ons-col>
            <v-ons-col width="130px">
              <v-ons-select id="prefecture" name="prefecture" v-model="selectedPrefecture">
                <option v-for="x in prefectures" :key="x.id" :value="x.id">{{x.name}}</option>
              </v-ons-select>
            </v-ons-col>
          </v-ons-row>
        </v-ons-list-item>
        <v-ons-list-item  modifier="nodivider">
          <div class="search-btn"><ons-button @click="search" class="ml10">検索</ons-button></div>
        </v-ons-list-item>
      </v-ons-list>
    </v-ons-modal>

    <v-ons-modal v-if="profileImageModalWindowVisible === true" :visible="profileImageModalWindowVisible" @click="profileImageModalWindowVisible = false">
      <img class="fullsize-profile-image" :src="`http://a1333.ml/img/profile/${modalItem.uuid}.jpg`">
    </v-ons-modal>

    <v-ons-modal v-if="profileDetailModalWindowVisible === true" :visible="profileDetailModalWindowVisible" @click="profileDetailModalWindowVisible = false" class="profile-modal">
      <h1>{{ modalItem.name }}</h1>
      <div class="user-info">
        <span>{{ `${getGenderString(modalItem.gender)} / ${modalItem.age}歳 / ${modalItem.prefecture}` }}</span>
      </div>
      <img class="halfsize-profile-image" :src="`http://a1333.ml/img/profile/${modalItem.uuid}.jpg`">
      <div class="user-info">
        <span>
          {{ modalItem.msg }}
        </span>
      </div>
      <ons-button v-if="$root.userId != modalItem.user_id" @click="showMessagePage(modalItem.user_id)"> <v-ons-icon icon="fa-envelope" /> メッセージを送る</ons-button>
    </v-ons-modal>
  </v-ons-page>
</template>

<script>
import MessagePage from '../Message/MessagePage'
import Enumerable from 'linq'
import vueSlider from 'vue-slider-component'

export default {
  components: {
    vueSlider
  },
  data () {
    return {
      users: [],
      searchModalWindowVisible: false,
      profileImageModalWindowVisible: false,
      profileDetailModalWindowVisible: false,
      modalItem: null,
      prefectures: [],
      selectedItem: -1,
      sliderRefreshed: false,
      gender: -1,
      selectedPrefecture: 0,
      ageValue: [
        20,
        29
      ],
      ageMin: 20,
      ageMax: 29,
      options: {
        width: '100%',
        height: 8,
        dotSize: 16,
        min: 20,
        max: 29,
        disabled: false,
        show: true,
        useKeyboard: true,
        tooltip: 'false',
        formatter: '{value}',
        enableCross: false,
        bgStyle: {
          'backgroundColor': '#fff',
          'boxShadow': 'inset 0.5px 0.5px 3px 1px rgba(0,0,0,.36)'
        },
        tooltipStyle: {
          'backgroundColor': '#666',
          'borderColor': '#666'
        },
        processStyle: {
          'backgroundColor': '#88d5ff'
        }
      }
    }
  },
  async mounted () {
    await this.getPrefectures()
  },
  watch: {
    ageValue (val) {
      this.ageMin = val[0]
      this.ageMax = val[1]
    },
    '$root.activeTabIndex' (value) {
      if (value === 1) {
        this.search()
      }
    }
  },
  methods: {
    searchModalWindow () {
      this.searchModalWindowVisible = true
      setTimeout(() => {
        this.refreshSlider()
      }, 500)
    },
    showMessagePage: function (userId) {
      if (this.$root.userId === null) {
        alert(this.$root.needRegisterMessage)
        this.$root.activeTabIndex = this.$root.UserProfileSettingTabIndex
      } else {
        this.$root.activeTabIndex = this.$root.messageTabIndex
        while (this.$root.messagePageStack.length > 1) {
          this.$root.messagePageStack.pop()
        }
        this.$root.messagePageStack.push({
          extends: MessagePage,
          data () {
            return {
              toUserId: userId
            }
          }
        })
      }
    },
    getGenderString: function (gender) {
      let genders = [ '男性', '女性', '非公開' ]
      return genders[gender]
    },
    showProfileImageModalWindow: function (userId) {
      let item = Enumerable.from(this.users)
        .where(x => x.user_id === userId)
        .first()
      this.modalItem = item
      this.profileImageModalWindowVisible = true
    },
    showProfileDetailModalWindow: function (userId) {
      let item = Enumerable.from(this.users)
        .where(x => x.user_id === userId)
        .first()
      this.modalItem = item
      this.profileDetailModalWindowVisible = true
    },
    async sendHeart (id) {
      if (this.$root.userId === null) {
        alert(this.$root.needRegisterMessage)
        this.$root.activeTabIndex = this.$root.UserProfileSettingTabIndex
      } else {
        await this.$root.postMessage(id, 'いいね！')
        alert('「いいね」しました。')
      }
    },
    async sendReport (userId, messageId) {
      if (this.$root.userId === null) {
        alert(this.$root.needRegisterMessage)
        this.$root.activeTabIndex = this.$root.UserProfileSettingTabIndex
      } else {
        let result = confirm('通報しますか？')
        if (result === true) {
          await this.$root.sendReport(userId, messageId)
          alert('通報しました。')
        }
      }
    },
    refreshSlider () {
      this.$refs.ageSelector.refresh()
      console.log('refreshed')
    },
    async getPrefectures () {
      let response = await fetch('http://a1333.ml/api/prefecture', { mode: 'cors', credentials: 'include' })
      let json = await response.json()
      this.prefectures = json
    },
    async search () {
      console.log('age min - ', this.ageMin)
      console.log('age max - ', this.ageMax)
      let response = await fetch('http://a1333.ml/api/search', {
        method: 'post',
        headers: { 'Content-type': 'application/json; charset=UTF-8' },
        mode: 'cors',
        credentials: 'include',
        body: JSON.stringify(
          {
            gender: this.gender,
            age_min: this.ageMin,
            age_max: this.ageMax,
            prefecture: this.selectedPrefecture
          })
      })
      let json = await response.json()
      console.log('json', json)
      if (json.length > 0) {
        this.users = json
        this.searchModalWindowVisible = false
      } else {
        alert('検索結果: 0件')
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div.toolbar-search-icon {
  margin-left: 10px;
}

.color-gray {
  color: gray;
}

.color-pink {
  color: palevioletred;
}

.fullsize-profile-image {
  width: 200px;
  height: 200px;
  border-radius: 100%;
}

.halfsize-profile-image {
  width: 100px;
  height: 100px;
  border-radius: 100%;
}

ons-list-title {
  text-transform: none;
}

ons-list-item {
  cursor: pointer;
}
.list-item {
  padding: 0;
}
.list-item__center, .gray-btns .left, .image-list {
  padding-left: 15px;
}
.label {
  padding-left: 15px;
}
.f-left {
  float: left;
}
.left-btn {
  border-radius: 3px 0 0 3px;
}
.right-btn {
  border-radius: 0 3px 3px 0;
}
.f-left .button {
  padding-left: 14px;
  padding-right: 14px;
  border: 1px solid #88d5ff;
  color: #212121;
  background: transparent;
}
.f-left .button.no-radius {
  border-radius: 0;
  border-left-color: transparent !important;
  border-right-color: transparent !important;
  padding-left: 13px;
  padding-right: 13px;
}
.f-left .button.active {
  background: #88d5ff;
  color: #fff;
}
.pt-5 {
  padding-top: 10px;
}
.modal-list {
  padding: 10px;
  margin: 24px;
  border-radius: 5px;
}
.search-btn {
  margin: 0 auto;
}
</style>
