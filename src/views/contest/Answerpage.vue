<template lang="pug">
div(color="#F7F3E8")
  v-container
    v-card(class="ma-4")
      //料理の名前を入れる##
      div.headline.ml-2.mb-1 {{ time }}
      div.display-1.ml-4.font-weight-bold お題: {{ resipetitle }}

      div.headline.ma-8.mb-0 料理名
      v-text-field.ma-8.mt-0(
        :counter="30"
        value=""
        v-model="name"
        :rules="[rules.required, rules.namemax]"
        label="料理名を入力！"
        required)

      div.headline.ma-8.mb-0 コメント
      v-textarea.ma-8.mt-0(
        :counter="140"
        value=""
        v-model="outline"
        :rules="[rules.required, rules.commentmax]"
        label="料理について自由に書いてください！")
        required

      div.headline.ma-8.mb-0 写真
      //input(
        type="file"
      //)
      v-file-input.ma-8.mt-0(
        @change="selectfile"
        accept="image/*"
        label="写真を選択してください！"
        )

      .text-center.pt-10.pb-12
        div(v-if="isLoggingin == true")
          div.red--text.mb-4 {{ error }}
          v-btn.title(color="#FFB618" @click="toquestion") 送信
        div(v-else)
          div コンテスト参加にはログインが必要です。
          v-btn.title(color="#FFB618" @click="tologin") ログイン
</template>

<script>
import Recipe from '..//components/Recipe'
import Materials from '..//components/Materials'
import Cookies from 'js-cookie'
import axios from 'axios'

export default{
  components: {
    Recipe,
    Materials
  },
  data: function(){
    return{
      time: "7",
      resipetitle:'',
      total_member: 1,
      uploadFile: null,
      url: "https://imgfp.hotp.jp/IMGH/21/64/P028842164/P028842164_480.jpg",
      name: "",
      outline: "",
      error: "",
      rules: {
        required: value => !!value || '入力必須項目です。',
        namemax: v => v.length <= 30 || '30文字以内でオナシャス',
        commentmax: v => v.length <= 140 || '140文字以内でオナシャス',
      },
    }
  },
  props:{
    isLoggingin: Boolean,
    userid: Number
  },
  created: function(){
    //useridをサーバーに送って投票状況を確認する？
  },
  computed: {
        contestid: function(){
            var contestid = Cookies.get('contestid')
            return contestid
        }
  },
  mounted: function(){
    let self = this
    //情報取得
    axios.get('http://localhost:8080/api/contest/info/'+String(self.contestid))
    .then(function (response) {
        var data = response.data
        console.log(data["title"])
        console.log(data["time"])
        console.log(data["votetime"])
        self.resipetitle = data["title"]
        self.time = data["time"]
    })
  },
  methods: {
    selectfile :function(e){
      console.log(e)
      var file = e.data
      this.uploadFile = file
    },
    toquestion: function(){
      let self = this
      if(this.name=="" || this.outline=="" || this.url==""){
        this.error="すべての項目を入力してから送信してください！"
      }else{
        this.error = ""
        axios.post('http://localhost:8080/api/contest/send',{   
            userid:this.userid,
            contestid:this.contestid,
            title: this.name,
            comment: this.outline,
            url: this.url
        })
      .then(function (response) {
        self.$router.push("/questionpage")
      })
      }
    },
    tologin: function(){
      this.$router.push("/loginpage")
    },
  }
}
</script>