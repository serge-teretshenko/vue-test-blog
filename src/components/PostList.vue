<template>
<div class="posts-wrapper">
   <div class="action-panel" v-show="this.actionPanel">
       <custom-button class="custom-button create-button" @click="toggleModalBox">Create Post</custom-button>

        <select class="sort-field" v-model="selectedSort" @change="sortPosts">
            <option value="" disabled selected>Sort by</option>
            <option value="title">Post Title</option>
            <option value="body">Post Text</option>  
            <option value="id">Post ID</option>
        </select> 

        <input class="search-field" type="text" placeholder="Search by Title" v-model="searchQuery">
    </div>

    <h3 class="posts-title">{{ pageTitle }}</h3>

    <div class='posts' v-if="posts.length > 0">
        <div class="post" v-for="post in filteredPosts" :key="post.id">            
            <h3 class="post__title">
                <span> Post {{ post.id }}: </span>
                {{ post.title }}
            </h3>
            <p class="post__text"> {{ post.body }} </p>

            <div class="post-buttons">
                <custom-button 
                    class="edit-button"
                    @click="$router.push(`/posts/${post.id}`)">
                        Open Post
                </custom-button>
                <custom-button @click="deletePost(post.id)" class='delete-button'></custom-button>
            </div>
        </div>
    </div>
    <div v-else>
        <no-page text="No Posts"></no-page>
    </div>

    <modal-box :show="isModalBoxShow" @hide="toggleModalBox">
        <PostForm @create="addPost" :post="post" />    
    </modal-box>

</div>
</template>

<script>
import CustomButton from '@/shared/Button'
import ModalBox from '@/shared/ModalBox'
import NoPage from '@/shared/NoPage'
import PostForm from '@/components/PostForm.vue'

export default {
    name: 'PostList',
    components: {
        PostForm,
        CustomButton,
        ModalBox,
        NoPage
    },
    props: { 
        postsPerPage: Number,
        actionPanel: Boolean,
        pageTitle: String
    },
    data() {
       return {
           posts: [],
           isModalBoxShow: false,
           post: {
               userId: 1,
               id: null,
               body: '',
               title: '',
           },
           selectedSort: '',
           searchQuery: '',
       }
    },
    watch: {
        
    },
    computed: {
        sortedPosts() {
            if (this.selectedSort === 'id') {
                return [...this.posts].sort((post1, post2) => post1.id - post2.id)
            } else {
                return [...this.posts].sort((post1, post2) => post1[this.selectedSort]?.localeCompare(post2[this.selectedSort]))
            }
        },               
        filteredPosts() {
            return this.sortedPosts.filter(post => post.title.toLowerCase().includes(this.searchQuery.toLowerCase()))
        }
    },
    methods: {
        async getPosts() {
            await fetch(`https://jsonplaceholder.typicode.com/posts?_limit=${this.postsPerPage}`)
                .then((response) => response.json())
                .then((posts) => this.posts = posts);
        },
        async addPost(post) {
            await fetch('https://jsonplaceholder.typicode.com/posts', {
                method: 'POST',
                body: JSON.stringify({
                    userId: 1,
                    title: post.title,
                    body: post.body,
                }),
                headers: {
                    'Content-type': 'application/json; charset=UTF-8',
                },
            })
            .then((response) => response.json())
            .then((newPost) => this.posts.unshift(newPost));
            this.toggleModalBox();
        },
        deletePost(id) {
            fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
                method: 'DELETE',
            });
            this.posts = this.posts.filter(post => post.id !== id)
        },
        toggleModalBox() {
            this.isModalBoxShow = !this.isModalBoxShow;
        },
    },
    mounted() {
        this.getPosts()
    }
}
</script>

<style lang="scss" scoped>
    .posts-wrapper {
        background-color: #ffffff;
        border: 10px solid #c3c3c3;
        padding: 5px 10px 10px;
    }
    .action-panel {
        display: flex;
        justify-content: space-between;
        margin-bottom: 25px;
    }
    .sort-field, .search-field {
        width: 125px;
        height: 36px;
        margin: 10px 0 10px;
        font-size: 14px;
        padding: 0 10px;
        box-sizing: border-box;
    }
    .posts-title {
        border-bottom: 1px solid #999999;
        text-transform: uppercase;
        text-align: center;
        margin: 15px 0;
    }
    .post {
        padding: 5px 0;
        margin-bottom: 15px;
        border-bottom: 1px solid #999;
        position: relative;

        &__title { 
            font-size: 20px;
            text-align: left;
            padding-right: 30px;
            
            span { 
                font-weight: 400;
                color: #a94443; }
            }
        
        &__text { 
            text-align: left;
        }
    }
    .post-buttons {
        text-align: right;
    }
    .custom-button {
        margin: 10px;
    }
    .delete-button {
        position: absolute;
        top: 0;
        right: 0;
        margin: 0;
    }
    @media screen and (max-width: 600px) {
        .action-panel {
            flex-direction: column;
            justify-content: left;
            padding: 15px;
        }
        .custom-button {
            margin: 0 0 15px;
        }
        .sort-field, .search-field { 
            width: 100%;
        }
    }
</style>