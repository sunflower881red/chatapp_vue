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
        <dl class="log_loop" v-for="log in chatlog" :key="log">
          <div class="name_iconbox">
            <div class="icon_cover animate__animated animate__headShake">
              <img :src="log.iconurl" alt="icon" class="icon">
            </div>
            <dt class="log_blank_box animate__animated animate__headShake">{{ log.name }}</dt>
          </div>
          <div class="log_arrow animate__animated animate__headShake">▲</div>
          <dd class="log_box animate__animated animate__headShake">{{ log.talk }}</dd>
        </dl>
      </div>
      <input type="text" id="twt" class="talk_box anim" placeholder="テキストを入力してください" maxlength="240">
      <button class="anim btn_bottom" @click="writedatabase">write</button>
    </div>
    <div v-show="(view == 4)" class="com_wrapper">
      <div class="testA">部屋名を入力して、部屋に入室する</div>
      <div>(指定した部屋がない場合、新たに作成)</div>
      <input v-model="roominp" type="text" id="roomno" class="input_box">
      <button class="btn anim" @click="roomin(roominp)">roomin</button>
      <button v-if="(UserData.room != null)" class="btn anim" @click="roomin(UserData.room)">前回入った部屋に入る</button>
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
import { doc,updateDoc,setDoc,getFirestore,query,orderBy,onSnapshot,collection,limit,getDoc} from "firebase/firestore";
import {getAuth,signOut,createUserWithEmailAndPassword,signInWithEmailAndPassword,onAuthStateChanged, updateProfile } from "firebase/auth";

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {

};
const app = initializeApp(firebaseConfig);
const firestore = getFirestore(app);



export default 
{
  data(){
    return{
      keytime:0,
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
      roomR:"",
      UserData:{},
      talkHis:""
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
        if(this.talkHis!=a){
        setDoc(doc(firestore,'roomdata/rooms/'+this.room,String(Date.now())), {
        date: t,
        name: n, 
        talk: a,
        uid: ""+this.uidG,
        jun: Date.now(),
        iconurl:"https://picsum.photos/id/"+this.UserData.icon+"/50/50"
        });
        this.talkHis=a;
        }
        return;
      },
      signout()
      {
        const auth = getAuth();
        signOut(auth).then(() => {
          this.loginfo="ログインしていません";
          this.room="";
          this.UserData=[];
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
            updateProfile(user, { displayName:name,photoURL:"" });
            this.name=name;
            setDoc(doc(firestore, "userdata",user.uid), {
            uid: ""+user.uid,
            user:name,
            jun: Date.now(),
            icon:Math.floor(Math.random()*999)
            });
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
      async GetHistory(){
        const docRef = doc(firestore, "userdata", this.uidG);
        const docSnap = await getDoc(docRef);
        if (docSnap.exists()) {
          console.log("Document data:", docSnap.data());
          this.UserData=docSnap.data();
        } else {
          // doc.data() will be undefined in this case
          console.log("No such document!");
        }
      },
      logS()
      {
        const auth = getAuth();
        onAuthStateChanged(auth, (user) => {
          if (user) {
            const uid = user.uid;
            this.uidG = user.uid;
            this.GetHistory();
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
        const q = query(collection(firestore,'roomdata/rooms/'+this.room), orderBy("jun","asc"),limit(1000));
        const snaps = onSnapshot(q, (snapshot) => {
          snapshot.docChanges().forEach((change) => {
            if (change.type === "added") {
                  this.chatlog.unshift(change.doc.data());
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
        console.log(this.room);
        this.room=roomno;
        if(roomno==""){
          this.room = String(Math.floor(Math.random()*9999999));
        }
        updateDoc(doc(firestore, "userdata",this.uidG),{
        room: String(this.room),
        jun: Date.now()
        });
        this.changeview(3);
        this.snap();
      }
  },
  created(){
    document.addEventListener("keypress", event => {
      if(event.code==='Enter'){
        console.log("Enter"+this.keytime);
        this.keytime++;
        if(this.view===3&&this.keytime>5){
          this.uploaddatabase();
          this.keytime=0;
        }
      }
      return;
    });
    document.addEventListener("keyup", event => {
      if(event.code==='Enter'){
        this.keytime=0;
      }
      return;
    });
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
  background-color: #2c2c2c;
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
.name_iconbox{
  display: flex;
  margin-top: 20px;
  margin-bottom: 10px;
  flex-direction: column;
  align-items: center;
}
.icon{
  width: 50px;
  height: 50px;
}
.icon_cover{
  width: 50px;
  height: 50px;
  border-radius: 100%;
  overflow: hidden;
  margin-bottom: 2px;
}
.talk_box{
  position: fixed;
  left: 0;
  bottom: 0;
  padding: 5px;
  width: 80vw;
  height: 60px;
  font-size: 1rem;
  border-top: solid 3px #2c2c2c;
  border-right: solid 1.5px #2c2c2c;
  background-color: #ffffff;
  z-index: 56;
}
.name{
  display: flex;
  justify-content: flex-start;
  text-align: end;
  align-items: flex-end;
}
.input_box{
  padding: 20px;
  width: 300px;
  height: 50px;
  font-size: 1.5rem;
  border: solid 1px #121212;
}
.log_box{
  width: 65vw;
  height: auto;
  background-color: #2c2c2c;
  color: #ffffff;
  display: flex;
  justify-content: center;
  text-align: center;
  align-items: center;
  border-radius: 30px;
  margin-top: 0px;
  margin-bottom: 0px;
  padding: 20px;
}
.log_blank_box{
  width: 20vw;
  height: 20px;
  background-color: #f7f7f7;
  color: #2c2c2c;
  border: solid 2px #2c2c2c;
  display: flex;
  justify-content: center;
  text-align: center;
  font-size: 12px;
  align-items: center;
  border-radius: 30px;
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
  height: 15px;
  font-size:30px;
  color: #2c2c2c;
  display: flex;
  justify-content: center;
  text-align: center;
  align-items: center;
  position: relative;
  left: -30px;
  bottom: -3px;
  transform: skew(-30deg, 20deg)scale(1, 1.2);
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
  border-top: solid 3px #2c2c2c;
  border-left: solid 1.5px #2c2c2c;
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
  border: double 20px #151515;
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
