<template>
<div class="bg">
  <div class="main_wrapeer"></div>
  <div class="ui_wrapeer">
  </div>
  <div class="ui_footer_wrapeer">
    <div v-if="(view != 3)" class="text">{{ loginfo }}</div>
    <div v-if="(view == 1)" class="com_wrapper">
      <div class="testA">Your Mail adress:</div>
      <input type="text" id="Lmail" class="input_box">
      <div class="testA">Your PassWord:</div>
      <input type="password" id="Lpass" class="input_box">
      <button class="btn anim" @click="login">Login</button>
      <p class="link_text" @click="changeview(2)">新規登録の方はこちら</p>
    </div>
  <div v-if="(view == 2)" class="com_wrapper">
    <div class="testA">Your Name:</div>
    <input type="text" id="name" class="input_box">
    <div class="testA">Your Mail adress:</div>
    <input type="text" id="mail" class="input_box">
    <div class="testA">Your PassWord:</div>
    <input type="password" id="pass" class="input_box">  
    <button class="btn anim" @click="FirstSignin">signin</button>
    <p class="link_text" @click="changeview(1)">ログインの方はこちら</p>
  </div>
    <div v-show="(view == 3)" class="com_wrapper">
      <p>部屋名:{{room}}</p>
      <button @click="signout" class="logout">logout</button>
      <div class="log_wrapper">
        <dl class="log_loop" v-for="(log,index) in chatlog" :key="log">
          <dt class="log_blank_box">{{ proflog[index] }}</dt>
          <div class="log_arrow">▲</div>
          <dd class="log_box">{{ log }}</dd>
        </dl>
      </div>
      <input type="text" id="twt" class="talk_box anim" placeholder="テキストを入力してください">
      <button class="anim btn_bottom" @click="writedatabase">write</button>
    </div>
    <div v-show="(view == 4)" class="com_wrapper">
      <div class="testA">部屋名を入力して、部屋に入室する</div>
      <div>(指定した部屋がない場合、新たに作成)</div>
      <input v-model="roominp" type="text" id="roomno" class="input_box">
      <button class="btn anim" @click="roomin(roominp)">roomin</button>
    </div>
  </div>
  <div class="main_wrapeer_C">
  </div>
  <!-- <div class="menu_wrapper" v-if="logflag">
    <div class="menu anim" @click="changeview(1)">Info</div>
    <div class="menu anim" @click="changeview(2)">Login</div>
    <div class="menu anim" @click="changeview(3)">Signup</div>
  </div> -->
</div>
</template>

<script>
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { doc,setDoc,getFirestore,query,addDoc,orderBy,onSnapshot,collection,limit,getDoc} from "firebase/firestore";
import {getAuth,signOut,createUserWithEmailAndPassword,signInWithEmailAndPassword,onAuthStateChanged, updateProfile } from "firebase/auth";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
/////////////////
};
const app = initializeApp(firebaseConfig);
const firestore = getFirestore(app);


export default 
{
  data(){
    return{
      no:0,
      roominp:"",
      room:"",
      name:"",
      mailG:"",
      uidG:"",
      view:1,
      loginfo:"ログインしていません",
      logflag:false,
      chatlog:[],
      proflog:[],
      roomR:""
    }
  },
  methods:{
    time(){
      this.no++;
      console.log(this.no);
    },
    uploaddatabase(){
        const date = new Date()
        const Y = date.getFullYear()
        const M = ("00" + (date.getMonth()+1)).slice(-2)
        const D = ("00" + date.getDate()).slice(-2)
        const h = ("00" + date.getHours()).slice(-2)
        const m = ("00" + date.getMinutes()).slice(-2)
        const s = ("00" + date.getSeconds()).slice(-2)
        var t = Y+"/"+M+"/"+D+"  "+h+":"+m+":"+s; 
        var a = document.getElementById("twt").value;
        var n = this.name;
        addDoc(collection(firestore,this.room), {
        date: t,
        name: n, 
        talk: a,
        uid: ""+this.uidG,
        jun: Date.now()
        });
        return;
      },
      signout()
      {
        const auth = getAuth();
        signOut(auth).then(() => {
          this.loginfo="ログインしていません";
          this.changeview(1);
        }).catch(() => {
          this.loginfo="ログアウトに失敗";
        });
      },
      FirstSignin()
      {
        const auth = getAuth();
        const email = document.getElementById( "mail" ).value;
        const pass = document.getElementById( "pass" ).value; 
        const name = document.getElementById( "name" ).value; 
        createUserWithEmailAndPassword(auth, email, pass)
          .then((userCredential) => {
            const user = userCredential.user;
            updateProfile(user, { displayName:name, photoURL: "" });
            this.name=name;
            console.log(user);
          })
          .catch(() => {
            this.loginfo="登録に失敗しました";
          });
      },
      login()
      {
        const auth = getAuth();
        const email = document.getElementById( "Lmail" ).value;
        const pass = document.getElementById( "Lpass" ).value; 
        signInWithEmailAndPassword(auth, email, pass)
          .then((userCredential) => {
            const user = userCredential.user;
            this.name=user.displayName;
            console.log(user);
          })
          .catch(() => {
            this.loginfo="ログインに失敗しました";
          });
      },
      logS()
      {
        const auth = getAuth();
        onAuthStateChanged(auth, (user) => {
          if (user) {
            const uid = user.uid;
            this.uidG = user.uid;
            this.dataR = getDoc(doc(firestore, "userdata",this.uidG));
            const mail = ""+user.email;
            this.mailG = mail.slice(0,mail.indexOf('@'));
            this.name=user.displayName;
            console.log(uid+"ログインに成功しました\n\n");
            this.loginfo="ログインに成功しました";
            this.changeview(4);
            // ...
          } 
        });
      },
      async snap()
      {
        let tk,ta;
        let index=0;
        const q = query(collection(firestore, this.room), orderBy("jun","asc"),limit(60));
        const snaps = onSnapshot(q, (snapshot) => {
          snapshot.docChanges().forEach((change) => {
            if (change.type === "added") {
                //tk = change.doc.get("date")+"\n"+change.doc.get("uid");
                tk = change.doc.get("name");
                ta = change.doc.get("talk");
                this.proflog.unshift(tk);
                this.chatlog.unshift(ta);
                console.log(this.chatlog+"_"+index);
            }
            if (change.type === "modified") {
                // tk = document.getElementById( "talkR" ).value;
                // ta = change.doc.get("name")+"の発言が変更された。"+"\n\n";
                // document.getElementById( "talkR" ).value=ta+tk;
            }
            if (change.type === "removed") {
                // tk = document.getElementById( "talkR" ).value;
                // ta = change.doc.get("name")+"の発言が削除された。"+"\n\n";
                // document.getElementById( "talkR" ).value=ta+tk;
            }
            index++;
          });
        });
        console.log(snaps);
      },
      changeview(no){
        this.view=no;
      },
      writedatabase(){
        this.uploaddatabase();
      },
      roomin(roomno){
        this.room=roomno;
        if(roomno==""){
          this.room = String(Math.floor(Math.random()*9999999));
        }
        setDoc(doc(firestore, "userdata",this.uidG), {
        room: String(this.room),
        uid: ""+this.uidG,
        jun: Date.now()
        });
        this.changeview(3);
        this.snap();
      }
  },
  created(){
      this.logS();     
  }
}
</script>

<style>
@import 'animate.css';
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@100&display=swap');
@import url('https://fonts.googleapis.com/css2?family=BIZ+UDGothic&display=swap');
body{
  font-family: 'Noto Serif JP', serif;
}
.logout{
  width: 100px;
  height: 30px;
  background-color: #121212;
  color: #ffffff;
  margin-top:5px;
}
textarea{
  background-color: #000000;
  color: white;
}
.link_text:hover{
  color: violet;
  cursor: pointer;
}
.link_text:active{
  color: violet;
  cursor: pointer;
}
p{
  margin-top: 40px;
}
.talk_box{
  position: fixed;
  left: 0;
  bottom: 0;
  padding: 5px;
  width: 80vw;
  height: 60px;
  font-size: 1rem;
  border: solid 3px #141414;
  background-color: #ffffff;
  z-index: 56;
}
.input_box{
  padding: 20px;
  width: 300px;
  height: 50px;
  font-size: 1.5rem;
  border: solid 1px #121212;
}
.log_box{
  width: 80vw;
  height: 100px;
  background-color: #2c2c2c;
  color: #ffffff;
  display: flex;
  justify-content: center;
  text-align: center;
  align-items: center;
  border-radius: 30px;
  margin-top: 0px;
  margin-bottom: 0px;
  outline: dashed 2px rgb(113, 113, 113);
}
.log_blank_box{
  width: 20vw;
  height: 50px;
  background-color: #2c2c2c;
  color: #ffffff;
  display: flex;
  justify-content: center;
  text-align: center;
  align-items: center;
  border-radius: 30px;
  margin-top: 40px;
  max-width: 200px;
  overflow: hidden;
}
.log_name{
  width: 20vw;
  height: 20px;
  color: #2c2c2c;
  display: flex;
  justify-content: start;
  text-align: center;
  align-items: center;
  margin-top: 20px;
  max-width: 200px;
  overflow: hidden;
}
.log_arrow{
  font-size:30px;
  color: #2c2c2c;
  display: flex;
  justify-content: start;
  text-align: center;
  align-items: center;
  position: relative;
  bottom: -15px;
  left: 30px;
  
}
.log_wrapper{
  margin-bottom: 100px;
}
.menu_wrapper{
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translate(-50%, 0);
  width: calc(100vw - 40px);
  display: flex;
  justify-content: space-between;
  z-index: 5;
}
.anim{
  transition: all 1s;
}
.menu{
  width: 100%;
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background-color: #000000;
  z-index: 6;
  padding: 0;
  margin: 0;
  color: #ffffff;
}
 .menu:hover{
  height: 60px;
  background-color: aliceblue;
  color: black;
 }
 .menu:active{
  height: 30px;
  background-color: rgb(0, 0, 0);
  color: rgb(255, 255, 255);
 }
.btn{
  width: 200px;
  height: 60px;
  margin-top: 50px;
  background-color: black;
  color: aliceblue;
}
.btn:hover{
  width: 200px;
  height: 60px;
  background-color: aliceblue;
  color: black;
  border: solid 1px black;
}
.btn:active{
  width: 180px;
  height: 40px;
  background-color: rgb(0, 0, 0);
  color: rgb(255, 255, 255);
  border: solid 1px rgb(255, 255, 255);
}
.btn_bottom{
  position: fixed;
  width: 20vw;
  height: 60px;
  margin: 0;
  right: 0;
  bottom: 0;
  z-index: 56;
  border:solid 2px black;
  background-color: rgb(117, 27, 27);
  color: aliceblue;
}
.btn_bottom:hover{
  height: 60px;
  background-color: aliceblue;
  color: black;
  border: solid 1px black;
}
.btn_bottom:active{
  height: 40px;
  background-color: rgb(0, 0, 0);
  color: rgb(255, 255, 255);
  border: solid 1px rgb(255, 255, 255);
}
.bottom{
  position: relative;
  bottom: 100px;
}
.main_wrapeer{
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  border: double 20px rgb(13, 13, 14);
  outline: solid 10px rgb(255, 255, 255);
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100vw;
  height: 100vh;
  background-color: rgb(255, 255, 255);

}
.ui_wrapeer{
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100vw;
  height: 100vh;
  z-index: 1;
}
.com_wrapper{
  display: flex;
  flex-direction: column;
  align-items: center;
}
.main_wrapeer_C{
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: calc(100vw - 40px);
  height: calc(100vh - 40px);
  background-color: rgb(255, 255, 255);
  z-index: 1;
}
.ui_footer_wrapeer{
  position: absolute;
  top: 20px;
  left: 20px;
  width: calc(100vw - 40px);
  height: calc(100vh - 40px);
  z-index: 2;
  -ms-overflow-style: none;
  scrollbar-width: none;
  overflow: scroll;
  overflow-x:hidden;
}
.ui_footer_wrapeer::-webkit-scrollbar{
  display: none;
}

.test{
  justify-content: center;
  display: flex;
  flex-direction: column;
  text-align: center;
  font-family: 'Noto Serif JP', serif;

}
.text{
  justify-content: center;
  display: flex;
  flex-direction: column;
  text-align: center;
  font-family: 'Noto Serif JP', serif;
  font-weight:lighter;
  font-size:1rem;
  color: rgb(20, 20, 20);
  margin-top:20px;
}
.testA{
  justify-content: center;
  display: flex;
  flex-direction: column;
  text-align: center;
  font-family: 'Noto Serif JP', serif;
  font-weight:lighter;
  font-size:1rem;
  color: rgb(20, 20, 20);
  padding: 20px;

}
.testB{
  margin-top: 50px;
  justify-content: center;
  display: flex;
  flex-direction: column;
  text-align: center;
  font-family: 'Noto Serif JP', serif;
  font-weight:lighter;
  font-size:12vw;
  color: rgb(20, 20, 20);
  padding: 30px;
}
</style>
