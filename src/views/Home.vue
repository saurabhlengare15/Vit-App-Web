<template>
  <div id="timeline" class="container justify-content-center">
    <input type="text" v-model="search" placeholder="Search Posts" class="form-control mb-4" />
    <div v-for="(post,index) in filteredPosts" :key="index">
      <div class="card mb-4">
        <div class="card-body">
          <div class="card-title container-fluid">
            <div class="row">
              <div class="col-sm-6 col-10 not">
                  Notice By <strong> {{post.from}} </strong>
              </div>
              <div v-if="loggedIn" class="col">
                <button type="button" class="close" @click="rmPost(post.id)">
                  <span class="cross">&times;</span>
                </button>
              </div>
            </div><hr class="line">
          </div>
          <div class="text-center">
            <img v-if="post.type=='image'" :src="post.mediaUrl" class="card-img img-fluid" />
          </div>
          <p class="card-text mt-3">{{post.notice}}</p>
          <a
            v-if="post.type=='pdf'"
            class="form-control bg-light"
            :href="post.mediaUrl"
            target="_blank"
          >Click to view PDF</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as firebase from "firebase/app";
import "firebase/firestore";
import "firebase/auth";
export default {
  data() {
    return {
      posts: [],
      search: "",
      loggedIn: false,
    };
  },
  methods: {
    getPosts(){
      firebase
      .firestore()
      .collection("timeline")
      .doc("all")
      .collection("timelinePost")
      .orderBy("timestamp", "desc")
      .get()
      .then((querySnapshot) => {
        this.posts = querySnapshot.docs.map((doc) => {
          return { id: doc.id, ...doc.data() };
        });
      })
      .catch((e) => console.log(e));
    },
    async rmPost(id) {
      if (confirm("Do you really want to delete?")) {
        await firebase
          .firestore()
          .collection("timeline")
          .doc("all")
          .collection("timelinePost")
          .doc(id)
          .delete();
          this.getPosts()
      }
    },
  },
  computed: {
    filteredPosts() {
      return this.posts.filter((post) => {
        return (
          post.from.toLowerCase().match(this.search.toLowerCase()) ||
          post.notice.toLowerCase().match(this.search.toLowerCase())
        );
      });
    },
  },
  created() {
    firebase.auth().onAuthStateChanged((user) => (this.loggedIn = !!user));
    this.getPosts();
    
  },
};
</script>

<style scoped>
#timeline {
  margin: 0 auto;
  max-width: 600px;
  position: relative;
  margin-top: 1rem;
  padding-bottom: 1rem;
}
a{
  text-decoration: none;
}
a:hover{
  color: #393185;
}
.cross{
  background-color: white;
  color: black;
}

.not{
  font-size: 16px;
  margin-left: -15px;
}
hr.line{
  border: 1px solid rgb(209, 209, 209);
  margin-top: 10px;
  margin-left: -35px;
  width: 114%;
}
</style>