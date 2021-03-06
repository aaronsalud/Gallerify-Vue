<template>

  <v-container class="text-xs-center">
    <!-- User Details Card -->
    <v-flex
      sm6
      offset-sm3
    >
      <v-card>
        <v-layout>
          <v-flex xs5>
            <v-img
              height="125px"
              contain
              :src="user.avatar"
            ></v-img>
          </v-flex>
          <v-flex xs7>
            <v-card-title primary-title>
              <div>
                <div class="headline">{{user.username}}</div>
                <div>Joined {{formatDate(user.joinDate)}}</div>
                <div class="hidden-xs-only font-weight-thin">{{user.favourites.length}} Favourites</div>
                <div class="hidden-xs-only font-weight-thin">{{userPosts.length}} Post Added</div>
              </div>
            </v-card-title>
          </v-flex>
        </v-layout>
      </v-card>
    </v-flex>

    <!-- Posts Liked by User -->
    <v-container v-if="!userFavourites.length">
      <v-layout
        row
        wrap
      >
        <v-flex xs12>
          <h2>You currently don't have any favourited posts.</h2>
        </v-flex>
      </v-layout>
    </v-container>
    <v-container
      class="mt-3"
      v-else
    >
      <v-flex xs12>
        <h2 class="font-weight-light">Favourited
          <span class="font-weight-regular">{{userFavourites.length}}</span>
        </h2>
      </v-flex>
      <v-layout
        row
        wrap
      >
        <v-flex
          xs12
          sm6
          v-for="favourite in userFavourites"
          :key="favourite._id"
        >
          <v-card
            class="mt-3 ml-1 mr-2"
            hover
          >
            <v-img
              height="30vh"
              :src="favourite.imageUrl"
              @click="goToPost(favourite._id)"
            ></v-img>
            <v-card-text>{{favourite.title}}</v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>

    <!-- Post Created by current user logged in -->
    <v-container v-if="!userPosts.length">
      <v-layout row>
        <v-flex xs12>
          <h2>You currently don't have any posts.</h2>
        </v-flex>
      </v-layout>
    </v-container>
    <v-container class="mt-3">
      <v-flex xs-12>
        <h2 class="font-weight-light">
          Your Posts
          <span>{{userPosts.length}}</span>
        </h2>
      </v-flex>
      <v-layout
        row
        wrap
      >
        <v-flex
          xs12
          sm6
          v-for="post in userPosts"
          :key="post._id"
        >
          <v-card
            class="mt-3 ml-1 mr-2"
            hover
          >
            <v-btn
              color="info"
              floating
              fab
              small
              dark
              @click="loadPost(post)"
            >
              <v-icon>edit</v-icon>
            </v-btn>
            <v-btn
              color="error"
              floating
              fab
              small
              dark
              @click="handleDeleteUserPost(post)"
            >
              <v-icon>delete</v-icon>
            </v-btn>
            <v-img
              height="30vh"
              :src="post.imageUrl"
              @click="goToPost(post._id)"
            ></v-img>
            <v-card-text>{{post.title}}</v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>

    <!-- Edit Post Modal -->
    <v-dialog
      xs12
      sm6
      offset-sm3
      persistent
      v-model="editPostDialog"
    >
      <v-card>
        <v-card-title class="headline grey lighten-2">Update Post</v-card-title>
        <!-- Add Post Form -->
        <v-container>
          <v-form
            v-model="isFormValid"
            lazy-validation
            ref="form"
            @submit.prevent="handleUpdateUserPost"
          >
            <!-- Title Input -->
            <v-layout row>
              <v-flex xs12>
                <v-text-field
                  :rules="titleRules"
                  v-model="title"
                  label="Post Title"
                  type="text"
                  required
                ></v-text-field>
              </v-flex>
            </v-layout>

            <!-- Image Url -->
            <v-layout row>
              <v-flex xs12>
                <v-text-field
                  :rules="imageRules"
                  v-model="imageUrl"
                  label="Image Url"
                  type="text"
                  required
                ></v-text-field>
              </v-flex>
            </v-layout>

            <!-- Image Preview -->
            <v-layout row>
              <v-flex xs12>
                <img
                  :src="imagePreviewUrl"
                  height="300px"
                >
              </v-flex>
            </v-layout>

            <!-- Categories Select -->
            <v-layout row>
              <v-flex xs12>
                <v-select
                  v-model="categories"
                  :rules="categoriesRules"
                  :items="['Art', 'Education', 'Travel', 'Photography', 'Technology', 'Food', 'Furniture']"
                  multiple
                  label="Categories"
                ></v-select>
              </v-flex>
            </v-layout>

            <!-- Description -->
            <v-layout row>
              <v-flex xs12>
                <v-textarea
                  :rules="descriptionRules"
                  v-model="description"
                  label="Description"
                  type="text"
                  required
                ></v-textarea>
              </v-flex>

            </v-layout>

            <v-divider></v-divider>

            <v-card-actions>

            </v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              type="submit"
              class="success--text"
              flat
              :disabled="!isFormValid"
            >Save</v-btn>
            <v-btn
              class="error--text"
              flat
              @click="editPostDialog = false"
            >Cancel</v-btn>

          </v-form>
        </v-container>
      </v-card>
    </v-dialog>

  </v-container>
</template>

<script>
import { mapGetters } from "vuex";
import moment from "moment";
export default {
  name: "Profile",
  data() {
    return {
      editPostDialog: false,
      isFormValid: true,
      title: "",
      imageUrl: "",
      imagePreviewUrl: "",
      categories: [],
      description: "",
      titleRules: [
        title => !!title || "Title is required",
        title =>
          title.length < 20 ||
          "Title must have less than 20 characters in length"
      ],
      imageRules: [image => !!image || "Image url is required"],
      categoriesRules: [
        categories =>
          categories.length >= 1 || "At least one category is  required"
      ],
      descriptionRules: [
        description => !!description || "Description is required",
        description =>
          description.length < 200 ||
          "Description must have less than 200 characters in length"
      ]
    };
  },
  computed: {
    ...mapGetters(["user", "userFavourites", "userPosts"])
  },
  watch: {
    imageUrl(value) {
      setTimeout(() => (this.imagePreviewUrl = this.imageUrl), 1000);
    }
  },
  created() {
    this.handleGetUserPosts();
  },
  methods: {
    goToPost(id) {
      this.$router.push(`/posts/${id}`);
    },
    handleGetUserPosts() {
      this.$store.dispatch("getUserPosts", {
        userId: this.user._id
      });
    },
    handleUpdateUserPost() {
      if (this.$refs.form.validate()) {
        // update user post
        this.$store.dispatch("updateUserPost", {
          postId: this.postId,
          userId: this.user._id,
          title: this.title,
          imageUrl: this.imageUrl,
          categories: this.categories,
          description: this.description
        });
        // Close modal
        this.editPostDialog = false;
      }
    },
    loadPost(
      { _id, title, imageUrl, categories, description },
      editPostDialog = true
    ) {
      this.editPostDialog = editPostDialog;
      this.postId = _id;
      this.title = title;
      this.imageUrl = imageUrl;
      this.categories = categories;
      this.description = description;
    },
    handleDeleteUserPost(post) {
      this.loadPost(post, false);
      const deletePost = window.confirm(
        "Are you sure you want to delete this post?"
      );
      if (deletePost) {
        this.$store.dispatch("deleteUserPost", {
          postId: this.postId
        });
      }
    },
    formatDate(date) {
      return moment(new Date(date)).format("ll");
    }
  }
};
</script>
