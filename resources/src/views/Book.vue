<template>
    <div>
        <Topbar></Topbar>
        <Sidebar title="Book details" />
        <Main>
            <div class="row mr-0">
                <div class="col-md-12">
                    <div v-if="loading">
                        Loading...
                    </div>
                    <div v-if="book.loaded">
                        <div class="row">
                            <div class="col-md-5">
                                <Book :book="book"></Book>
                                <WriteReview :book="book" />
                                <BookReviews :book="book" />
                            </div>
                            <div class="col-md-7">
                                <UsersBooking v-if="!user.isModerator" :book="book" />
                                <ModeratorsBooking v-if="user.isModerator" :book="book" />
                            </div>
                        </div>
                    </div>
                    <div v-if="!loading && !book.loaded">
                        <h1>Book not found</h1>
                    </div>
                </div>
            </div>
        </Main>
    </div>
</template>

<script>
import Main from '@/components/Main';
import Topbar from '@/components/Topbar';
import Sidebar from '@/components/Sidebar';
import Book from '@/components/Book';
import WriteReview from '@/components/WriteReview';
import BookReviews from '@/components/BookReviews';
import UsersBooking from '@/components/users/Bookings';
import ModeratorsBooking from '@/components/moderators/Bookings';
import { mapState } from 'vuex';

export default {
    components: {
        Main,
        Topbar,
        Sidebar,
        Book,
        WriteReview,
        BookReviews,
        UsersBooking,
        ModeratorsBooking
    },

    data () {
        return {
            loading: true,
            book: {
                loaded: false,
                id: '',
                title: '',
                author: '',
                details: '',
                isbn: '',
                categories: []
            }
        };
    },

    computed: {
        ...mapState([
            'user'
        ])
    },

    mounted () {
        this.book.id = this.$route.params.id;
        this.$http.get(`/books/${this.book.id}`).then((response) => {
            const book = response.data.book;
            this.book.title = book.title;
            this.book.author = book.author;
            this.book.details = book.details;
            this.book.image = book.image;
            this.book.isbn = book.ISBN;
            this.book.categories = book.categories;
            this.book.rating = book.rating;

            this.loading = false;
            this.book.loaded = true;
        }).catch(() => {
            this.loading = false;
            this.book.loaded = false;
        });
    }
};
</script>
