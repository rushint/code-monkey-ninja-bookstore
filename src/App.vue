<template>
  <v-app>
    <v-navigation-drawer
      fixed
      :mini-variant="miniVariant"
      :clipped="clipped"
      v-model="drawer"
      app
    >
        <v-list>
            <v-subheader>Development Topics</v-subheader>
            <v-list-tile 
                value="true"
                v-for="(topic, i) in topicItems"
                :key="i" 
                v-on:click="fetchBooks(topic)"
            >
                <v-list-tile-action>
                    <v-icon v-html="topic.icon" v-bind:color="topic.headerColor"></v-icon>
                </v-list-tile-action>
                <v-list-tile-content>
                    <v-list-tile-title v-text="topic.title"></v-list-tile-title>
                </v-list-tile-content>
            </v-list-tile>

            <v-divider></v-divider>

            <v-subheader>Favorites</v-subheader>
            <v-list-tile 
                value="true"
                v-for="(fav, i) in favoriteItems"
                :key="i" 
                 v-on:click.stop="showFavBook(fav)"
            >
                <v-list-tile-action>
                    <img class="book-small-thumb" v-bind:src="fav.img"/>
                </v-list-tile-action>
                <v-list-tile-content>
                    <v-list-tile-title v-text="fav.title"></v-list-tile-title>
                </v-list-tile-content>
                <v-list-tile-action v-on:click.stop="removeFav(fav)">
                    <v-icon>clear</v-icon>
                </v-list-tile-action>
            </v-list-tile>
        </v-list>
    </v-navigation-drawer>

    <v-toolbar fixed app dark color="primary" :clipped-left="clipped">
        <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
        <v-btn flat class="toolbar__title" v-on:click="goHome">{{title}}</v-btn>
        <!-- <v-toolbar-title v-text="title"></v-toolbar-title> -->
    </v-toolbar>

    <v-content>
        <v-container fluid>
            <template v-if="!hasBooks">
                <v-slide-y-transition mode="out-in">
                    <v-layout column align-center>
                        <img src="/public/code-monkey.jpg" alt="Code Monkey Ninja" v-bind:class="{ animate: loading }" class="mb-5 logo" />
                        <blockquote>
                            &#8220;Achievement, is just another skill of a ninja.&#8221;
                            <footer>
                                <small>
                                    <em>&mdash;Jarius Raphel</em>
                                </small>
                            </footer>
                        </blockquote>
                    </v-layout>
                </v-slide-y-transition>
            </template>

            <template v-else>
                <v-layout row>
                    <v-flex xs12>
                        <v-card>
                            <v-toolbar v-bind:color="headerColor" dark>
                                <v-toolbar-title>You search for &quot;{{searchTerm}}&quot;</v-toolbar-title>
                                <v-spacer></v-spacer>
                                <v-toolbar-title>returned {{totalItems}} results</v-toolbar-title>
                            </v-toolbar>

                            <v-list three-line>
                                <div v-for="(book, ndx) in books" :key="ndx">
                                    <v-list-tile ripple @click="navigateTo(book.volumeInfo.infoLink)">
                                        <img class="book-thumb" v-if="book.volumeInfo.imageLinks" v-bind:src="book.volumeInfo.imageLinks.thumbnail"/>

                                        <v-list-tile-content>
                                            <v-list-tile-title v-html="book.volumeInfo.title"></v-list-tile-title>
                                            <v-list-tile-sub-title v-html="book.volumeInfo.publisher"></v-list-tile-sub-title>
                                            <v-list-tile-sub-title v-html="book.volumeInfo.description"></v-list-tile-sub-title>
                                        </v-list-tile-content>

                                        <v-list-tile-action v-on:click.stop="toggleFavorite(book)">
                                            <v-icon
                                                color="yellow lighten-1"
                                                v-if="book.selected"
                                            >
                                                star
                                            </v-icon>
                                            <v-icon
                                                color="grey darken-2"
                                                v-else
                                            >
                                                star_border
                                            </v-icon>
                                        </v-list-tile-action>
                                    </v-list-tile>

                                    <v-divider v-if="ndx + 1 < books.length" :key="book.volumeInfo.title"></v-divider>
                                </div>
                            </v-list>
                        </v-card>
                    </v-flex>
                </v-layout>

                <div class="text-xs-center">
                    <v-pagination v-on:input="navPage" :length="pagination.total" v-model="pagination.page" :total-visible="5"></v-pagination>
                </div>
            </template>
        </v-container>
    </v-content>

    <v-footer :fixed="fixed" app>
      <span>&copy; 2017</span>
    </v-footer>
  </v-app>
</template>

<script>
    import axios from 'axios';

    var STORAGE_KEY = 'vue-js-book-store'

    var favStorage = {
        fetch: function () {
            var favs = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
            return favs;
        },
        save: function (favs) {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(favs));
        }
    }

    export default {
        data () {
            return {
                clipped: false,
                drawer: true,
                fixed: false,
                topicItems: [
                    { icon: 'class', title: 'Angular', subFilter: 'javascript', headerColor: 'deep-orange' },
                    { icon: 'class', title: 'Bootstrap', subFilter: 'css', headerColor: 'green' },
                    { icon: 'class', title: 'Cordova', subFilter: 'apache', headerColor: 'deep-purple' },
                    { icon: 'class', title: 'CSS', subFilter: '', headerColor: 'green' },
                    { icon: 'class', title: 'Ionic', subFilter: 'javascript', headerColor: 'deep-orange' },
                    { icon: 'class', title: 'HTML', subFilter: '', headerColor: 'deep-orange' },
                    { icon: 'class', title: 'React', subFilter: 'javascript', headerColor: 'deep-orange' },
                    { icon: 'class', title: 'Vue', subFilter: 'javascript', headerColor: 'deep-orange' },
                    { icon: 'class', title: 'Webkit', subFilter: '', headerColor: 'deep-orange' }
                ],
                favoriteItems: favStorage.fetch(),
                miniVariant: false,
                right: true,
                rightDrawer: false,
                title: 'Code Monkey Ninja Book Store',
                errors: null,
                loading: false,
                pagination: {
                    page: 1,
                    limit: 12,
                    total: 0
                },
                hasBooks: false,
                searchTerm: '',
                subFilter: '',
                books: [],
                headerColor: '',
                selTopic: null
            }
        },
        created() {
//console.log('created() - this.favoriteItems: ', this.favoriteItems);
        },
        watch: {
            favoriteItems: {
                handler: function(favs) {            
                    favStorage.save(favs);
                }
            }
        },
        methods: {
            goHome: function() {
                this.hasBooks = false;
                this.books = [];
                this.headerColor= '';
                this.termChanged = true;
                this.searchTerm = '';
                this.subFilter = '';
            },
            fetchBooks: function(topic, startNdx = 0) {
                let termChanged = false;
                let searchTerm = topic.title;
                let subFilter = topic.subFilter;

                this.selTopic = topic;
                this.loading = true;
                this.headerColor = topic.headerColor;

                const GOOGLEBOOK_URL = 'https://www.googleapis.com/books/v1/volumes?q='+ searchTerm + '+' + subFilter +'&maxResults=' + this.pagination.limit + '&startIndex=' + startNdx;

                if(searchTerm != this.searchTerm) {
                    termChanged = true;
                }
                axios.get(GOOGLEBOOK_URL)
                    .then(response => {
                        // JSON responses are automatically parsed.
                        this.books = response.data.items;

                        if(termChanged) {
                            this.totalItems = response.data.totalItems
                        }

                        // Set the selected flag in the response.data based on 
                        // items in the favoriteItems array 
                        if(this.favoriteItems) {
                            this.books.forEach(function(bookObj) {
                                let favItems = this.favoriteItems;
                                let bookId = bookObj.id;

                                favItems.forEach(function(obj, ndx) {
                                    if(bookId == obj.id) {
                                        bookObj.selected = true;
                                    }
                                }, this);
                            }, this);
                        }

                        this.hasBooks = true;
                        this.searchTerm = searchTerm;
                        this.subFilter = subFilter;

                        if(termChanged) {
                            this.pagination.page = 1;
                        }

                        this.pagination.total = Math.ceil(response.data.totalItems / this.pagination.limit);

                        this.loading = false;
                    })
                    .catch(e => {
                        this.hasBooks = false;

                        this.errors.push(e)
                    }
                )
            },
            navigateTo: function(url) {
                window.open(url, '_blank');
            },
            // Show next set of books - pagination
            navPage: function(payload) {
                this.fetchBooks(this.selTopic, (payload * this.pagination.limit) - this.pagination.limit);
                this.pagination.page = payload;
            },
            // Remove favorite book from local storage and list
            removeFav: function(fav) {
                let favNdx = 0;

                for(let ndx = 0; ndx < this.favoriteItems.length; ndx++) {
                    if(this.favoriteItems[ndx].id == fav.id) {
                        favNdx = ndx;
                    }
                }

                // Change the selected property in the books array to false
                if(this.books.length > 0) {
                    this.books.forEach(function(obj, ndx) {
                        if(fav.id == obj.id) {                           
                            this.books[ndx].selected = false;
                        }
                    }, this);
                }

                this.favoriteItems.splice(favNdx, 1);
            },
            // Add book as favorite to local storage and list
            addFav: function(newFav) {
                this.favoriteItems.push({
                    id: newFav.id,
                    img: newFav.volumeInfo.imageLinks.thumbnail, 
                    title: newFav.volumeInfo.title, 
                    publisher: newFav.volumeInfo.publisher, 
                    description: newFav.volumeInfo.description,
                    infoLink: newFav.volumeInfo.infoLink
                });
            },
            // Toggle book as favorite
            toggleFavorite: function(fav) {
                let favObj = this.favoriteItems.find(o => o.id === fav.id);

                if(favObj) {
                    // TODO 3: find the matching fav in the favoriteItems array and pass it to removeFav

                    this.removeFav(favObj);
                } else {
                    this.addFav(fav);

                    fav.selected = true;
                }
            },
            // Show book detail page in new tab/window
            showFavBook: function(book) {
                this.navigateTo(book.infoLink)
            }
        }
    }
</script>

<style scoped>
    .logo {
        width: 200px;
    }
    .book-thumb {
        height: 75px; 
        margin-right: 10px;
    }
    .book-small-thumb {
        height: 50px; 
        margin-right: 10px;
    }

    @keyframes zoominoutsinglefeatured {
        0% {
            transform: scale(1,1);
        }
        50% {
            transform: scale(1.2,1.2);
        }
        100% {
            transform: scale(1,1);
        }
    }

    .animate {
        animation: zoominoutsinglefeatured 1s alternate infinite;
    }
</style>

