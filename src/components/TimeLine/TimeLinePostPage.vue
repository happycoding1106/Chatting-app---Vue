<template>
  <ons-page>
    <v-ons-toolbar>
      <div class="left">
        <v-ons-back-button>戻る</v-ons-back-button>
      </div>
    </v-ons-toolbar>
    <div class="pa-5">
      <span>メッセージ</span>
    </div>
    <div class="pa-5">
      <textarea id="msg" class="textarea" rows="3" v-model="msg"></textarea>
    </div>

    <p class="pa-5">
      <ons-button @click="postToTimeLine">タイムラインに投稿する</ons-button>
    </p>
  </ons-page>
</template>

<script>
export default {
  data () {
    return {
      msg: null
    }
  },
  methods: {
    postToTimeLine: async function () {
      let response = await fetch('http://a1333.ml/api/timeline', {
        method: 'post',
        headers: { 'Content-type': 'application/json; charset=UTF-8' },
        mode: 'cors',
        credentials: 'include',
        body: JSON.stringify(
          {
            msg: this.msg
          })
      })
      let json = await response.json()
      if (json.success) {
        await this.$root.getTimeLineItems()
        this.$root.timeLinePageStack.pop()
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
textarea {
  width: 100%;
  height: 200px;
}
.pa-5 {
  padding: 10px;
}
.page__content {
  padding: 10px;
}
.mt-10 {
  margin-top: 10px;
}
</style>
