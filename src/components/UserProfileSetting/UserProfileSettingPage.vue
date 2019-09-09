<template>
  <ons-page>
    <form id="form1">
      <div>
        <vueCropper
          ref="cropper"
          :img="vueCropperOption.img"
          :outputSize="vueCropperOption.size"
          :outputType="vueCropperOption.outputType"
          :autoCrop="vueCropperOption.autoCrop"
          :autoCropWidth="vueCropperOption.autoCropWidth"
          :autoCropHeight="vueCropperOption.autoCropHeight"
          :fixed="vueCropperOption.fixed"
          :fixedNumber="vueCropperOption.fixedNumber"
          style="height: 300px"
          class="image-cropper"
        ></vueCropper>
        <div class="center crop-btns">
          <label class="btn" for="image"><v-ons-button style="margin: 6px 0">プロフィール画像アップロード</v-ons-button></label>
          <v-ons-button @click="changeScale(1)" class="bt-3">+</v-ons-button>
          <v-ons-button @click="changeScale(-1)" class="bt-3">-</v-ons-button>

          <input type="file" id="image" accept="image/*" style="position:absolute; clip:rect(0 0 0 0);" @change="chooseImg($event)">
        </div>
      </div>

      <table>
        <tr>
          <td><label for="name">ニックネーム</label></td>
          <td><v-ons-input id="name" class="custom-input" name="name" modifier="underbar" v-model="name" /></td>
        <tr>
          <td><label for="age">年齢</label></td>
          <td>
            <vue-slider v-model="age" v-bind="options" ref="ageSelector"></vue-slider>
          </td>
        </tr>
        <tr>
          <td><label for="prefecture">都道府県</label></td>
          <td>
            <v-ons-select id="prefecture" name="prefecture" v-model="prefecture">
              <option v-for="x in $root.prefectures" :key="x.id" :value="x.id">{{x.name}}</option>
            </v-ons-select>
          </td>
        </tr>
        <tr>
          <td><label for="gender">性別</label></td>
          <td>
            <v-ons-col>
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
          </td>
        </tr>
        <tr>
          <td><label for="comment">ひとこと</label></td>
          <td><v-ons-input id="comment" class="custom-input" name="comment" modifier="underbar" maxlength=100 v-model="comment" /></td>
        </tr>
      </table>

      <p class="btn-wrapper">
        <ons-button @click="signup" class="user-register-btn">登録</ons-button>
      </p>
    </form>
  </ons-page>
</template>

<script>
// import ImageCropPage from '../components/ImageCropPage'
import vueSlider from 'vue-slider-component'
import VueCropper from 'vue-cropper'

export default {
  components: {
    vueSlider,
    VueCropper
  },
  data () {
    return {
      name: null,
      age: 20,
      prefecture: null,
      gender: 0,
      comment: null,
      vueCropperPreview: {},
      sliderRefreshed: false,
      vueCropperOption: {
        img: null,
        size: 1,
        full: false,
        outputType: 'jpg',
        canMove: true,
        fixedBox: false,
        original: false,
        canMoveBox: true,
        autoCrop: true,
        autoCropWidth: 200,
        autoCropHeight: 200,
        fixed: true,
        fixedNumber: [1, 1]
      },
      options: {
        width: '100%',
        height: 8,
        dotSize: 16,
        min: 20,
        max: 65,
        disabled: false,
        show: true,
        useKeyboard: true,
        tooltip: 'always',
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
  watch: {
    '$root.activeTabIndex' (value) {
      if (value === 3 && !this.sliderRefreshed) {
        setTimeout(() => {
          this.refreshSlider()
        }, 500)
        this.sliderRefreshed = true
      }
    }
  },
  async mounted () {
    await this.$root.getMyInfo()
    await this.$root.getPrefectures()

    this.name = this.$root.userInfo.name
    this.age = this.$root.userInfo.age
    this.prefecture = this.$root.userInfo.prefecture
    this.gender = this.$root.userInfo.gender
    this.comment = this.$root.userInfo.comment

    if (this.$root.userInfo.uuid !== null) {
      let response = await fetch(`http://a1333.ml/img/profile/${this.$root.userInfo.uuid}.jpg`)
      let blob = await response.blob()
      console.log(blob)
      this.vueCropperOption.img = window.URL.createObjectURL(blob)
      console.log('img loaded')
    }
  },
  methods: {
    refreshSlider () {
      this.$refs.ageSelector.refresh()
      console.log('refreshed')
    },
    async getCroppedBlob () {
      let data = await new Promise(resolve => this.$refs.cropper.getCropBlob((data) => { resolve(data) }))
      return data
    },
    startCrop () {
      this.$refs.cropper.startCrop()
    },
    changeScale (num) {
      num = num || 1
      this.$refs.cropper.changeScale(num)
    },
    chooseImg (e) {
      let file = e.target.files[0]
      if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
        return false
      }

      let reader = new FileReader()
      reader.onload = (e) => {
        if (typeof e.target.result === 'object') {
          this.vueCropperOption.img = window.URL.createObjectURL(new Blob([e.target.result]))
        } else {
          this.vueCropperOption.img = e.target.result
        }
        // this.startCrop()
      }
      reader.readAsArrayBuffer(file)
    },
    signup: async function () {
      if (!document.getElementById('name').value) {
        alert('名前と年齢は必須です')
        return
      }
      if (this.$root.userId === null) {
        let data = new FormData(document.getElementById('form1'))
        if (this.vueCropperOption.img !== null) {
          data.append('profile_image', await this.getCroppedBlob())
        }
        let response = await fetch('http://a1333.ml/api/signup', {
          method: 'post',
          mode: 'cors',
          credentials: 'include',
          body: data
        })
        let json = await response.json()
        if (json.success) {
          this.$root.userId = json.user_id
          this.$root.setPushToken()
          alert('プロフを登録しました')
        }
      } else {
        let data = new FormData(document.getElementById('form1'))
        if (this.vueCropperOption.img !== null) {
          data.append('profile_image', await this.getCroppedBlob())
        }
        let response = await fetch('http://a1333.ml/api/update_profile', {
          method: 'post',
          mode: 'cors',
          credentials: 'include',
          body: data
        })
        let json = await response.json()
        if (json.success) {
          this.$root.userId = json.user_id
          this.$root.setPushToken()
          alert('プロフを更新しました')
        }
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
img {
  width: 100px;
  height: 100px;
}
.crop-btns {
  text-align: center;
  padding: 10px;
  border-top: 1px solid #eee;
}
table {
  width: 100%;
  padding: 0 12px;
}
table td {
  padding: 8px 0;
}
ons-input, ons-radio, ons-checkbox, ons-search-input {
  width: 100%;
}
p.btn-wrapper {
  text-align: center;
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
.vue-slider-component {
  padding: 0 !important;
}
</style>
