<template>
    <div>
        <h2>Articles</h2>
        <div class="card card-body mb-2" v-for="article in articles" :key="article.id">
            <h3>{{ article.title }}</h3>
            <p>{{ article.body }}</p>
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
        this.fetchArticles();
    },
    methods: {
        fetchArticles(page_url){//for requests we will use the fetch api. The page_url is an optionally argument, it will be needed for pagination.
        let vm = this;
            page_url = page_url || '/api/articles'; //page_url will be = page_url if there is one, OR it will be = '/api/articles'

            fetch(page_url)//TODO if there is this fetch thingy, then we don't need axios?

                .then(res => res.json())//take the received results, and with the arrow function return trick, return these results, transformed, mapped into json object? OK, from the api, we will receive an object. This object will contain three arrays. The first array, called data will contain all the articles. The meta and the links arrays are for pagination purposes. Here we need only the data array. So we need to acces res.data.

                .then(res => {//here we are getting the actual data data from the api
                    this.articles = res.data;//TODO is it possible to replace this with axios???
                    vm.makePagination(res.meta, res.links);//res.meta, res.links are the arrays from res, recevied from the api
                })
                .catch(err => console.log(error));
        }
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