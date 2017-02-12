<template>
  <div class="posts-container">
    <div class="ui fixed inverted menu">
        <div class="right menu">
          <div class="ui right aligned category search item">
            <div class="ui transparent icon input inverted">
              <input type="text" placeholder="Filter by title or tag..." v-model="filterValue">
              <i class="filter link icon"></i>
            </div>
            <div class="results"></div>
          </div>
          <a href="#" class="item" v-on:click.stop.prevent="newPost()">
            <div class="ui icon" data-tooltip="Create a post" data-position="bottom right">
              <i class="add large icon"></i>
            </div>
          </a>
        </div>
    </div>

    <div class="ui basic modal">
      <div class="ui icon header">
        <i class="trash icon"></i>
        Delete the post "{{ postToDelete.title }}"
      </div>
      <div class="content">
        <p>Are you sure you want to delete this post?</p>
      </div>
      <div class="actions">
        <div class="ui red basic cancel inverted button">
          <i class="remove icon"></i>
          No
        </div>
        <div class="ui green ok inverted button">
          <i class="checkmark icon"></i>
          Yes
        </div>
      </div>
    </div>

    <div class="posts ui container centered">
      <div class="ui link cards centered aligned">
        <div class="ui card" v-bind:class="getPostStatusColor(post)"  v-for="post in posts">
          <div class="content">
            <div class="header">{{post.title}}</div>
            <div class="meta">
              <span class="right floated time">{{ publishedAt(post) }}</span>
              <span class="status">{{ status(post) }}</span>
            </div>
          </div>
          <div class="image cover">
            <div :style="'background-image: url(\''+ postCoverUrl(post) +'\')'">
            </div>
          </div>
          <div class="extra content">
            <i class="right floated large edit icon" v-on:click="navigateToPost(post)"></i>
            <i class="right floated large trash link icon" v-on:click="displayConfirmMessage(post)"></i>
            <div class="author">
              <img class="ui avatar image" :src="post.author.avatar_url"> {{post.author.name || post.author.login}}
            </div>
          </div>
          <div class="extra content">
            <a class="ui tiny label" v-for="tag in post.tags">
              <i class="tag icon"></i>
              {{ tag }}
            </a>
            <div class="" v-if="!post.tags || !post.tags.length">
              No tag
            </div>

          </div>
        </div>

    </div>
</div>
</template>

<script>
import moment from 'moment'
import uuid from 'node-uuid'

export default {
  name: 'posts',
  data: function() {
    return {
      postToDelete: {},
      filterValue: ''
    }
  },
  beforeCreate: () => {},
  mounted: function() {
    $('.ui.basic.modal')
      .modal({
        closable  : false,
        onDeny    : function(){
          console.log('deny')
        },
        onApprove : function() {
          console.log('approved')
        }
      })
  },
  methods: {
    status: function(post) {
      return !!post.published && `Published` || 'Draft';
    },
    publishedAt: function(post) {
      return !!post.published && moment(post.published_at).fromNow() || '';
    },
    postCoverUrl: function(post) {
      return post.image || 'http://hubpress.io/img/logo.png'
    },
    getPostStatusColor: function(post) {
      if (!post.original) {
        return 'red'
      } else if (post.original.content !== post.content) {
        // if post has changed
        return 'orange'
      } else {
        return 'green'
      }
    },
    displayConfirmMessage: function (post) {
      this.postToDelete = post
      $('.ui.basic.modal')
        .modal('show')
      ;
    },
    navigateToPost: function(post) {
      // named route
      this.$router.push({ name: 'post', params: { id: post._id }})
    },
    newPost: function() {
      // Create a new post
      this.$router.push({ name: 'post', params: { id: uuid.v4() }})
    },
    getPostTags: function(post) {
      return post.tags || []
    }

  },
  computed: {
    posts: function() {
      const filter = this.filterValue.trim()
      if (filter === '')
        return this.$store.state.hubpress.posts

      return this.$store.state.hubpress.posts.filter(post => {
        console.log(post)
        return (post.title.toLowerCase().indexOf(filter.toLowerCase()) >= 0) ||
          (post.tags && post.tags.filter(tag => tag.toLowerCase().indexOf(filter.toLowerCase()) >=0).length)
      })
    }
  }
}
</script>

<style>
.posts-container .menu {
  min-height: 47px;
}
.posts.ui.container {
  padding-top: 60px;
}

.image.cover > div {
  width: 100%;
  height: 200px;
  background-size: cover;
  background-repeat: no-repeat;
  background-position: 50% 50%;
}

@media screen and (max-width: 622px) {
  .posts.ui.container .ui.card .image.cover {
    display:none;
  }
}
</style>
