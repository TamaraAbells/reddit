<template>
  <div>
    <NotFound v-if='!community' resource='community' />
    <div v-if='community'>
      <CommunityHeader
        :border='true'
        :community='community'
      />
      <v-container>
        <v-row>
          <v-col cols='12' :md='8'>
            <CreatePostHeader v-if='isAuthenticated' :community='community.name' />
            <PostFilter :sort='sort' @selectSort='selectSort' />
            <PostList v-if='posts' :posts='posts' @vote='vote' />
            <!-- <Spinner v-if='!posts' /> -->
          </v-col>
          <v-col :md='4' v-if='$vuetify.breakpoint.mdAndUp'>
            <CommunityInfo :community='community' />
          </v-col>
        </v-row>
      </v-container>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import CommunityInfo from '@/components/Communities/Info.vue'
import CommunityHeader from '@/components/Communities/Header.vue'
import CreatePostHeader from '@/components/Posts/CreatePostHeader.vue'
import PostFilter from '@/components/Posts/PostFilter.vue'
import PostList from '@/components/Posts/PostList.vue'
// import Spinner from '@/components/Core/Spinner.vue'
import communities from '@/assets/json/communities.json'
import axios from 'axios'
import { calculateVote } from '@/utils.js'
import NotFound from '@/components/Core/NotFound.vue'

export default {
  name: 'Community',
  components: {
    CreatePostHeader,
    PostFilter,
    PostList,
    CommunityInfo,
    CommunityHeader,
    NotFound
    // Spinner
  },
  data: function () {
    return {
      communities,
      sort: 'Best',
      posts: [null, null, null, null, null]
    }
  },
  methods: {
    selectSort (sort) {
      this.sort = sort
      this.getPosts()
    },
    vote (data) {
      calculateVote(this.posts.find(p => p._id === data.postId), data.type)
      axios.post(`/api/posts/${data.type}`, {
        postId: data.postId
      })
    },
    getPosts () {
      this.posts = [null, null, null, null, null]
      axios.get('/api/posts', {
        params: {
          communityId: this.community.id,
          sort: this.sort
        }
      })
        .then(res => {
          if (res.data.success) {
            this.posts = res.data.posts.map(e => {
              const community = this.communities.find(c => c.id === e.communityId)

              return {
                ...e,
                communityName: community.name
              }
            })
          }
        })
    }
  },
  computed: {
    isAuthenticated () {
      return this.$store.state.isAuthenticated
    },
    community () {
      return communities.find(e => e.name === this.$route.params.community)
    }
  },
  watch: {
    community () {
      this.getPosts()
    }
  },
  mounted () {
    if (this.community) {
      this.getPosts()
    }
  }
}
</script>
