<template>
    <div class="review">
      <div class="review-title-stars">
          <h2 class="review-title">Clients Reviews</h2>
          <div class="stars"></div>
      </div>
      <div class="loading" v-if="!commentsList">Loading...</div>
      <div class="review-btn-comments">
            <span class="review-btn-msg" v-if="showReviewBtn" @click="displayComment = !displayComment">{{reviewMsg}}</span>
        <div action="" class="review-submit-review" v-if="displayComment" >
            <div class="review-container">
                <span class="review-container-msg" role="textbox" contenteditable="true">
                </span>
                <i class="icon-send" @click="postComment"></i>
            </div>
        </div>
        <div class="review-comments">
            <div class="reviews-items" v-for="review in commentsList" :key="review.id" :id="review.id.slice(15,32)">
                <div class="review-item" >
                    <i class="review-user-icon icon-avatar"></i>
                    <span class="" style="display:flex; align-self: center;">{{review.username}}</span>
                </div>
                <!-- <span class="review-date">{{review.date.toDate().toString().slice(0,21)}}</span> -->
                <div class="review-comment">
                    <div class="review-comment-container" :id="review.id.slice(6,30)">
                        <span class="review-comment-msg " role="textbox" :contenteditable="editContent"
                        :id="review.id"
                        @keyup.esc="escape(review.id)" >
                            {{review.comment}}
                        </span>
                        <span
                        class="review-dots" 
                        v-if="getUsername == review.username" 
                        @click="toggleVisibility(review.id.slice(2,23))" style="cursor: pointer"
                        :id="review.id.slice(7,28)">
                        ...
                        </span>
                        <i class="icon-send" @click="updateComment(review.id)" v-if="dispIcon == review.id"></i>
                    </div>
                    <label for="" class="review-escape-msg"
                     v-if="escapeMsg == review.id && cancel">
                        Press esc to 
                        <span style="color: blue; cursor: pointer" @click="escape(review.id)">cancel</span>
                    </label>
                    <div class="review-display-content" :id="review.id.slice(2,23)" style="display: none">
                        <ul class="review-display-items">
                            <li class="review-display-item" @click="edit(review.id)">Edit</li>
                            <li class="review-display-item" @click="removeComment(review.id)">Delete</li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
      </div>
  </div>
</template>


<script>
import dbase from '../assets/firebaseConfig/firebaseInit'
import firebase from 'firebase/app'
import 'firebase/auth'
import {mapGetters, mapState} from 'vuex'
import {v4 as uuidv4} from 'uuid'
export default {
    name: "Review",


    props:{
        prodId: String,
    },


    data(){
        return {
            dispIcon: false,
            showReviewBtn: false,
            displayComment: false,
            commentsList: [],
            productReviews: [],
            editContent: false,
            displayContentMenu: true,
            review: '',
            escapeMsg: false,
            cancel: false
        }
    },


    methods:{
        postComment(){
            console.log("posting comment...")
            const span = document.querySelector(".review-container-msg");
            console.log(span.innerHTML)
            if(span.innerHTML === ""){
                return;
            }
            const reviewId = uuidv4();
            const data = {
                id: reviewId,
                prodId: this.prodId,
                username: this.getUsername,
                // date: firebase.firestore.FieldValue.serverTimestamp(),
                comment: span.innerHTML,
                stars: 0
            };
            if(this.commentsList.length){
                dbase.collection("reviews").doc(this.prodId).update({
                    commentsList: firebase.firestore.FieldValue.arrayUnion(data)
                }).then(x => console.log("last review ", x))
            }else{
                dbase.collection("reviews").doc(this.prodId).set({commentsList:[data]})
            }
            this.commentsList =  [...this.commentsList, data]
            this.displayComment = false;
        },

        updateComment(id){
            const span = document.getElementById(id)
            const comment = span.innerHTML;
            const reviewId = uuidv4();
            const data = {
                id: reviewId,
                prodId: this.prodId,
                username: this.getUsername,
                // date: firebase.firestore.FieldValue.serverTimestamp(),
                comment: comment,
                stars: 0
            };
            let docRef = dbase.collection("reviews").doc(this.prodId);
            docRef.get().then(snapShot => {
                let itemToUpdate = snapShot.data().commentsList.find(item => item.id === id)
                if(!itemToUpdate){
                    return
                }
                itemToUpdate.comment = comment;
                itemToUpdate.id = reviewId;
                docRef.update({
                    commentsList: [...snapShot.data().commentsList.filter(item => item.id !== id),
                    itemToUpdate]
                });
                console.log("Comment updated!")
            })
            span.innerHTML = '';
            this.displayComment = false;
            const el = document.getElementById(id.slice(15,32));
            el.remove();
            this.commentsList =  [...this.commentsList, data]

        },

        removeComment(id){
            let docRef = dbase.collection('reviews').doc(this.prodId);
            docRef.get().then(snapShot => {
                let itemToUpdate = snapShot.data().commentsList.find(item => item.id === id);
                if(!itemToUpdate){
                    return;
                }
                docRef.update({
                    commentsList: firebase.firestore.FieldValue.arrayRemove(itemToUpdate)
                });
                console.log('comment removed!')
            })
            const element = document.getElementById(id.slice(15,32));
            element.remove();
            this.commentsList = this.commentsList.filter(el => el.id !== id)
        },

        edit(id){
            const id2 = id.slice(2,23);
            const id3 = id.slice(6,30);
            const id4 = id.slice(7,28);
            this.review = document.getElementById(id).innerHTML;
            this.dispIcon = id;
            this.editContent = true;
            this.displayContentMenu = false;
            var el1 = document.getElementById(id);
            var el2 = document.getElementById(id2);
            var el3 = document.getElementById(id3);
            var el4 = document.getElementById(id4);
            el1.classList.add("review-container-msg")
            el2.style.display = 'none';
            el3.classList.add('edit-active');
            el4.classList.add('deactive');
            this.escapeMsg = id;
            this.cancel = true;
        },

        toggleVisibility(id){
            const el = document.getElementById(id);
            if(el.style.display == "block"){
                el.style.display = "none"
            }else{
                el.style.display = "block"
            }
        },

        escape(id, val=true){
            if(val){
                const el = document.getElementById(id);
                el.innerHTML = this.review;
                el.classList.remove('review-container-msg');
            }
            const id3 = id.slice(6,30);
            const el3 = document.getElementById(id3);
            el3.classList.remove('edit-active');
            const id4 = id.slice(7,28);
            const el4 = document.getElementById(id4);
            el4.classList.remove('deactive');
            this.editContent = false;
            this.dispIcon = false;
            this.cancel = false;
        },

        retrieveReviews(){
            let list = [];
            dbase.collection("reviews").doc(this.prodId)
            .get()
            .then((querySnapshot) => {
                if(!querySnapshot.data()){
                    return
                }
                querySnapshot.data().commentsList.forEach((review) => {
                    console.log(review)
                    list.push(review);
                })
                this.commentsList = list
            })
        }
    },


    computed:{
        ...mapGetters({
            getUsername: 'getUsername'}),
        ...mapState({
            comments: "comments"
        }),
        reviewMsg(){
            if(!this.displayComment)
                return "Add review +"; 
            return "Hide review"
        },

 
    },


    watch:{
            prodId: function(){
                this.commentsList = [];
                dbase.collection('reviews').onSnapshot(snap => {
                    let rawDocs = snap.docChanges();
                    rawDocs.forEach(rawDoc => {
                        if((rawDoc.type === 'added' || rawDoc.type === 'modified') && rawDoc.doc.id === this.prodId){
                            let val = rawDoc.doc.data();
                            console.log("retrieving reviews", val);
                            this.commentsList = val.commentsList;
                        }
                    })
                })
            }
    },


    created(){
        if(this.getUsername){
            firebase.auth().currentUser.getIdTokenResult(true)
            .then(idTokenResult => {
                if(idTokenResult.claims.registered){
                    this.showReviewBtn = true;
                }
            })
            .catch(e => {
                console.log(e);
            })
        }
    },


    mounted(){
        this.retrieveReviews();
    }
}
</script>


<style >

.review{
    display: flex;
    flex-direction: row;
    border-top: 1px solid #ccc;
    padding-top:30px;
    margin: 0 0px 0 0px;
    width: 73%;
}
.review-title-stars{
    display: flex;
    flex-direction: column;
    width: 400px;

}
.review-title{
    text-align: left;
    margin-top: 0;
}
.review-btn-comments{
    display: flex;
    flex-direction: column;
}
.review-items{
    display: flex;
    flex-direction: column;
}
.review-item{
    display: flex;
    flex-direction: row;
}
.review-date{
    display: flex;
    color: grey;
    font-size: 12px;
}
.review-comment{
    display: flex;
    flex-direction: row;
    position: relative;
}
.review-display-content{
    position: absolute;
    left: 100px;
    top: 56px;
}
.review-dots{
    display: flex;
    align-self: center;
    margin-left: 10px;
}
.review-dots.deactive{
    display: none;
}
.review-display-items{
    display: flex;
    flex-direction: column;
    list-style: none;
    padding: 10px 0px 0px 0px;
    border: 1px solid black;
    border-radius: 5px;
    background-color: white;
  -moz-box-shadow:    3px 3px 5px 6px #ccc;
  -webkit-box-shadow: 3px 3px 5px 6px #ccc;
  box-shadow:         3px 3px 3px 3px #ccc;
}
.review-display-item{
    padding-bottom: 10px;
    width: 200px;
    text-align: left;
    padding-left: 10px;
}
.review-display-item:hover{
    background-color:#ccc;
    cursor: pointer;
}
.review-comment-msg{
    border: 1px solid #ccc;
    border-radius: 25px;
    margin: 20px 0 20px 0;
    padding: 20px 10px 20px 10px;
    text-align: left;
    max-width: 400px;
    min-width: 100px;
}
.review-comment-msg.review-container-msg{
    min-width: 100px;
    border: none;
}
.review-comment-container{
    display: flex;
}
.review-comment-container.edit-active{
    border: 1px solid #ccc;
    border-radius: 25px;
    margin: 20px 0 20px 0;
    padding-right: 25px;
}
.comment{
    max-width: 600px;
}
.review-user-icon{
    color: black;
}
.review-escape-msg{
    position: absolute;
    bottom: 0;
    font-size: 12px;
    margin-bottom: 6px;
    left: 19px
}
.review-container{
  border: 1px solid  rgb(88, 86, 86);
  border-radius: 25px;
  width: 650px;
  display: flex;
  padding-bottom: 20px;
}
.review-container-msg{

  font-family: inherit;
  font-size: inherit;
  padding: 10px 10px 10px 20px;
  box-sizing: border-box;
  min-width: 600px;
  max-width: 600px;
  outline: none;
  text-align: left;
  cursor: text;
}
.review-container-msg {
  display: block;
  width: 100%;
  overflow: hidden;
  min-height: 40px;
  line-height: 20px;
}
.review-container-msg[contenteditable]:empty::before {
  content: "Your comment ...";
  color: rgb(58, 55, 55);
}
.review-btn-msg{
    width: 95px;
    text-align: left;
    text-decoration: underline;
    cursor: pointer;
    color: purple;
    margin-bottom: 20px;
}
.icon-send{
    display: flex;
    align-self:flex-end;
    padding-bottom: 10px;
    cursor: pointer;
}
/****************************************************************************** */
@media only screen 
and (min-device-width: 320px) 
and (max-device-width: 414px)

and (-webkit-min-device-pixel-ratio: 2) {
.review{
    display: grid;
    place-items: center;
    margin-bottom: 50px;
}
.review-title-stars{
    width: 181px;
}
}
/****************************************************************************** */
@font-face {
  font-family: 'icomoon';
  src:  url('../assets/fonts/icomoon.eot?jvjlod');
  src:  url('../assets/fonts/icomoon.eot?jvjlod#iefix') format('embedded-opentype'),
    url('../assets/fonts/icomoon.ttf?jvjlod') format('truetype'),
    url('../assets/fonts/icomoon.woff?jvjlod') format('woff'),
    url('../assets/fonts/icomoon.svg?jvjlod#icomoon') format('svg');
  font-weight: normal;
  font-style: normal;
  font-display: block;
}

[class^="icon-"], [class*=" icon-"] {
  /* use !important to prevent issues with browser extensions that change fonts */
  font-family: 'icomoon' !important;
  speak: never;
  font-style: normal;
  font-weight: normal;
  font-variant: normal;
  text-transform: none;
  line-height: 1;

  /* Better Font Rendering =========== */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-paper-plane:before {
  content: "\e901";
}
.icon-travel:before {
  content: "\e901";
}
.icon-plane:before {
  content: "\e901";
}
.icon-user1:before {
  content: "\e900";
}
.icon-profile:before {
  content: "\e900";
}
.icon-avatar:before {
  content: "\e900";
}
.icon-arrow_drop_down:before {
  content: "\e902";
}
.icon-send:before {
  content: "\e903";
}
.icon-cart:before {
  content: "\e93a";
}
</style>