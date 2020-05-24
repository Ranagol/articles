<template>
    <div>
        <h2>Articles</h2>

        <!-- ADD/EDIT NEW ARTICLE -->
        <form @submit.prevent="addArticle" class="mb-3">
            <div class="form-group">
                <input v-model="article.title" type="text" class="form-control" placeholder="Title">
            </div>
            <div class="form-group">
                <textarea v-model="article.body" type="text" class="form-control" placeholder="Body"></textarea>
            </div>
            <button type="submit" class="btn btn-light btn-block">Save</button>
        </form>

        <!-- PAGINATION -->
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <!-- PREVIOUS -->
                <li v-bind:class = "[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a></li><!-- We want this class to be disabled, if there is no previous page -->

                <!-- CURRENT PAGE -->
                <li class="page-item disabled"><a class="page-link text dark" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a></li>
                <!--disabled in the page-item disabled: because this is disable link all the time. The link function is disabled. We use this only for displaying something like this: Page 2 of 6.  -->

                <!-- NEXT -->
                <li v-bind:class = "[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a></li><!-- We want this class to be disabled, if there is no previous page -->
            </ul>
        </nav>

        <!-- ARTICLES LIST -->
        <div class="card card-body mb-2" v-for="article in articles" :key="article.id">
            <h3>{{ article.title }}</h3>
            <p>{{ article.body }}</p>
            <hr>
            <!-- EDIT ARTICLE -->
            <button @click="editArticle(article)" class="btn btn-warning mb-2">Edit</button><!-- Here we are passing the entire article, not just the id number -->
            <!-- DELETE ARTICLE -->
            <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Articles',
    data() {
        return {
            articles: [],
            article: {
                id: '',
                title: '',
                body: '',

            },
            article_id: '',
            pagination: {},
            edit: false,//this is our edit state, false by default. We will use the same form for add and edit.
        }
    },
    created(){
        this.fetchArticles();//this function will fetch articles
    },
    methods: {
        fetchArticles(page_url){//for requests we will use the fetch api. The page_url is an optionally argument, it will be needed for pagination. There may be a page_url argument, or not. When the page is created, there will be no argument. When the next or previous button in the pagination bar is clicked, then, there will be an argument. page_url in that case will be either pagination.prev_page_url or pagination.next_page_url.
        let vm = this;
            page_url = page_url || '/api/articles'; //page_url will be = page_url if there is one, OR it will be = '/api/articles'

            fetch(page_url)//TODO if there is this fetch thingy, then we don't need axios?

                .then(res => res.json())//take the received results, and with the arrow function return trick, return these results, transformed, mapped into json object? OK, from the api, we will receive an object. This object will contain three arrays. The first array, called data will contain all the articles. The meta and the links arrays are for pagination purposes. Here we need only the data array. So we need to acces res.data.

                .then(res => {//here we are getting the actual data data from the api
                    this.articles = res.data;//TODO is it possible to replace this with axios???
                    vm.makePagination(res.meta, res.links);//res.meta, res.links are the arrays from res, recevied from the api
                })
                .catch(err => console.log(error));
        },
        makePagination(meta, links){//this is for creating the pagination
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev,
            }
            this.pagination = pagination;
        },
        deleteArticle(id){
            if(confirm('Are you sure?')){
                fetch(`api/article/${id}`, {
                    method: 'delete'//fetch here is used for deleting. This is why it is stated that the method is 'delete'.
                })
                .then( res => res.json())//TODO I am not really sure what is happening here with the .json thingy. I need to check this part with console.dir.
                .then( data => {
                    alert('Article deleted.');
                    this.fetchArticles();// I guess after deleting this is the way how we refresh the page
                })
                .catch(err => console.log(err));
            }
        },
        addArticle(){//addArticle() will add and update articles (two things!)
            if(!this.edit){
                //Add new article
                fetch('api/article', {
                    method: 'post',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then( res => res.json())
                .then( data => {
                    this.article.title = '';
                    this.article.body = '';
                    alert('Article added.');
                    this.fetchArticles();
                })
                .catch( err => console.log(err));
            } else {
                //Update article
                 fetch('api/article', {
                    method: 'put',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then( res => res.json())
                .then( data => {
                    this.article.title = '';
                    this.article.body = '';
                    alert('Article updated.');
                    this.fetchArticles();
                })
                .catch( err => console.log(err));
            }
        },
        editArticle(article){
            this.edit = true;
            this.article.id = article.id;
            this.article.article_id = article.id;
            this.article.title = article.title;
            this.article.body = article.body;
        }
        //TODO how to disable this shit picture on my desktop...
    }
}



//when we are fetching data from the api, we have to fetch the pagination data too.
/*THIS IS HOW OUR API-PAGINATION DATA LOOKS LIKE:

"links": {
"first": "http://127.0.0.1:8000/api/articles?page=1",//this will be the first page link
"last": "http://127.0.0.1:8000/api/articles?page=6",//this will be the last page link
"prev": null,//...since we will be at the start on the first page, and there is no zero page
"next": "http://127.0.0.1:8000/api/articles?page=2"//the next page will be page 2, since at the beginning we will be on page one
},

"meta": {
"current_page": 1,
"from": 1,
"last_page": 6,
"path": "http://127.0.0.1:8000/api/articles",
"per_page": 5,
"to": 5,
"total": 30
}

*/
</script>