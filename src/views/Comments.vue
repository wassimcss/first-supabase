<template>
  <div class="bg-white">
    <div class="row g-5 g-xxl-8">
      <div class="col-lg-8 p-5">
        <div
          class="card mb-5 mb-xxl-8"
          v-for="(comment, index) in commentsData"
          :key="index"
        >
          <div class="pb-0">
            <div class="d-flex align-items-center mb-5">
              <div class="d-flex align-items-center flex-grow-1">
                <div class="symbol symbol-45px me-5 rounded-circle">
                  <img
                   :src="require('../assets/images/profile.png')"
                    alt=""
                    class="rounded-circle"
                    width="60"
                  />
                </div>

                <div class="d-flex flex-column">
                  <a
                    href="#"
                    class="text-gray-800 text-hover-primary fs-6 fw-bolder"
                    >{{ comment.name }}</a
                  >
                </div>
              </div>
            </div>

            <div class="mb-5">
              <div v-if="editing !== index" class="cursor-pointer">
                <div
                  class="item"
                  effect="dark"
                  content="Click to Edit"
                  placement="top"
                >
                  <p
                    class="text-gray-800 fw-normal mb-5"
                    @click="enableEditing(index)"
                  >
                    {{ comment.desc }}
                  </p>
                </div>
              </div>
              <div v-if="editing === index">
                <input
                  type="textarea"
                  autosize
                  v-model="comment.desc"
                  class="p-2 text-gray-500 focus:outline-none"
                />

                <div class="editing-btns">
                  <span
                    class="svg-icon svg-icon-1 me-2"
                    @click="saveEditComment"
                  >
                    <!-- <inline-svg src="media/svg/shapes/check.svg" /> -->
                  </span>
                  <span
                    class="svg-icon svg-icon-1"
                    @click="cancelEditing(index)"
                  >
                    <!-- <inline-svg src="media/svg/shapes/cancel.svg" /> -->
                  </span>
                </div>
              </div>
            </div>

            <div class="separator mb-4"></div>
          </div>
        </div>

        <div>
          <textarea
            type="textarea"
            v-model="tempValue"
            cols="60"
            :autosize="{ minRows: 4 }"
            placeholder="Please input"
            ref="inputComment"
          />
          <br />

          <button
            @click="addComment"
            class="mr-6 mt-6 py-2 px-6 rounded-sm self-start text-sm
            text-white bg-at-light-green duration-200 border-solid
            border-2 border-transparent hover:border-at-light-green hover:bg-white
            hover:text-at-light-green"
            :disabled="isDisable"
          >
            Add
          </button>
          <span
            @click="cancelAddComment"
            class="mt-6 py-2 px-6 rounded-sm self-start text-sm
            text-white bg-at-light-green duration-200 border-solid
            border-2 border-transparent hover:border-at-light-green hover:bg-white
            hover:text-at-light-green"
          >
            Cancel
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from "vue";
import { supabase } from "../supabase/init";
import store from "../store";

export default {
  name: "comments",
  components: {},
  props: {
    // commentsData: Array,
  },
  setup() {
    const editing = ref(false);
    const tempValue = ref("");
    const user = computed(() => store.state.user);
    const commentsData = ref([]);

    const inputComment = ref();
    const previousComment = ref();
    const comments = commentsData;
    const commentSubscription = ref(undefined);

    const cancelEditing = (index) => {
      comments[index].desc = previousComment.value;
      editing.value = false;
    };
    const saveEditComment = () => {
      editing.value = false;
    };
    const addComment = async () => {
      const newComment = {
        name: user.value.email.substring(0, 4),
        //   date: "0 min",
        //   photo: "media/avatars/150-8.jpg",
        desc: tempValue.value,
      };

      //   context.emit("addNewComment", newComment);

      //   cancelAddComment();
      try {
        const { error } = await supabase.from("comments").insert([newComment]);
        if (error) throw error;
        tempValue.value = "";
      } catch (error) {
        alert(error.message);
      }
    };

    const cancelAddComment = () => {
      inputComment.value = "";
    };
    const enableEditing = (index) => {
      previousComment.value = comments[index].desc;

      editing.value = index;
    };
    const isDisable = computed(() => {
      const disable = tempValue.value.length ? false : true;
      return disable;
    });

    const watchComments = async () => {
      console.log("testttttt");
      try {
        commentSubscription.value = await supabase
          .from("comments")
          .on("*", (payload) => {
            if (payload.new) {
              commentsData.value = [...commentsData.value,payload.new];
            }
            console.log(commentsData.value, payload.new);
          })
          .subscribe();
        console.log(commentSubscription.value);
      } catch (error) {
        console.error(error);
      }
    };

    const getComments = async () => {
      try {
        const { data: comments, error } = await supabase
          .from("comments")
          .select("*");
        if (error) throw error;
        commentsData.value = comments;
      } catch (error) {
        console.error(error.message);
      }
    };

    onMounted(() => {
      getComments();
      watchComments();
    });

    const unsbscribeComments = async() => {
        await supabase.removeSubscription()
    }

    onUnmounted(() => {
        unsbscribeComments()
        console.log("Unsbscription")
    })
    

    return {
      tempValue,
      isDisable,
      cancelEditing,
      addComment,
      inputComment,
      enableEditing,
      editing,
      saveEditComment,
      cancelAddComment,
      commentsData,
    };
  },
};
</script>
<style lang="scss" scoped>
.editing-btns {
  display: flex;
  justify-content: flex-end;
  span:hover {
    cursor: pointer;
    filter: brightness(0.5);
  }
}
textarea {
  margin-top: 10px;
  margin-left: 50px;
  width: 500px;
  height: 100px;
  -moz-border-bottom-colors: none;
  -moz-border-left-colors: none;
  -moz-border-right-colors: none;
  -moz-border-top-colors: none;
  background: none repeat scroll 0 0 rgba(0, 0, 0, 0.07);
  border-color: -moz-use-text-color #ffffff #ffffff -moz-use-text-color;
  border-image: none;
  border-radius: 6px 6px 6px 6px;
  border-style: none solid solid none;
  border-width: medium 1px 1px medium;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.12) inset;
  color: #555555;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 1em;
  line-height: 1.4em;
  padding: 5px 8px;
  transition: background-color 0.2s ease 0s;
}

textarea:focus {
  background: none repeat scroll 0 0 #ffffff;
  outline-width: 0;
}
</style>
