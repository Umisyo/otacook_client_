<template>
<v-card>
<form>
    <v-text-field
        :rules="[rules.required]"
        v-model="email"
        class="mx-4 pt-5"
        label="ユーザーID"
        required
    ></v-text-field>
    <v-text-field
        :rules="[rules.required]"
        :type="passwordarea ? 'text' : 'password'"
        v-model="password"
        class="mx-4"
        label="パスワード"
        hint="八文字以上の文字列です."
        required
        @click:append="passwordarea = !passwordarea"
    ></v-text-field>
    <div class="text-right mr-4 red--text"> {{errors}} </div>
    <v-layout justify-space-around="">
        <v-layout class="tosignup">
            <div class="moji ml-4">会員登録をされていない方</div>
            <v-btn class="ma-4" @click="tosignup">サインアップページへ</v-btn>
        </v-layout>
        <v-btn class="ma-4" @click="submit">ログイン</v-btn>
    </v-layout>
</form>
</v-card>
</template>

<script>
import axios from 'axios'

export default {
    data () {
        return {
            errors: "",
            email: "",
            password: "",
            userid: -1,
            sessionid: -1,
            passwordarea: "",
            rules: {
                required: value => !!value || '入力必須',
            },
        }
    },
    methods: {
        tosignup: function(){
            this.$router.push("/signup")
        },
        submit: function(){
            let self = this
            if(this.email == "" || this.password == ""){
                this.errors = "入力していない項目があります"
            }else{
                //ここでメールアドレスとパスワードをサーバーに送って
                //sessionidとuseridを取得する
                //data="email="+this.email+"&pass"
                axios.post('http://localhost:8080/api/login',{
                    email:this.email,
                    password:this.password
                })
                .then(function (response) {
                    console.log(response.data);
                    if(response.data == "/USER/"){
                        self.errors = "ユーザーIDを確認"
                    }else if(response.data == "/PASS/"){
                        self.errors = "パスワードを確認"
                    }else{
                        //var data = JSON.parse(response.data)
                        var data = response.data
                        //console.log(data["userid"])
                        //console.log(data["sessionid"])
                        self.userid = Number(data["userid"])
                        self.sessionid = Number(data["sessionid"])
                        self.$emit('signin', self.sessionid, self.userid)
                        self.$router.back()
                    }
                })
            }
        },
    },
    props: {
        redirectto: String,
    },
}
</script>

<style scoped>
.moji{
    position: relative;
    top: 23px
}
</style>