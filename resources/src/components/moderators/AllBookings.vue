<template>
    <div>
        <h3>All bookings</h3>
        <div class="card">
            <div class="card-body pending-booking">
                <div class="card-text">
                    <h1>Pending for reviews</h1>
                    <hr />
                    <div v-if="pending.length === 0">No pending booking request yet.</div>
                    <div v-for="booking in pending" :key="booking.id">
                        <div class="clearfix">
                            <div class="float-left">
                                {{booking.user.name}} requested for {{ booking.quantity }} cop{{ booking.quantity > 1 ? 'ies' : 'y' }} of <router-link :to="'/books/' + booking.book.id">{{ booking.book.title }}</router-link>
                            </div>
                            <div class="float-right">
                                <button class="btn btn-sm btn-success" @click="approve(booking.id)">Approve</button>
                                <button class="btn btn-sm btn-danger" @click="cancelBooking(booking.id)" >Cancel</button>
                                <button class="btn btn-sm btn-primary" @click="update(booking.id, booking.quantity + 1)">Add one</button>
                                <button class="btn btn-sm btn-primary" @click="update(booking.id, booking.quantity - 1)">Remove one</button>
                            </div>
                        </div>
                        <hr />
                    </div>
                </div>
            </div>

            <div class="card-body approved-booking">
                <div class="card-text">
                    <h1>Approved</h1>
                    <hr />
                    <div v-if="approved.length === 0">No approved booking request yet.</div>
                    <div v-for="booking in approved" :key="booking.id" class="clearfix">
                        <div class="float-left">
                            {{ booking.quantity }} cop{{ booking.quantity > 1 ? 'ies' : 'y' }} of
                            <router-link :to="'/books/' + booking.book.id">{{ booking.book.title }}</router-link> is approved for {{booking.user.name}}
                        </div>
                        <hr />
                    </div>
                </div>
            </div>

            <div class="card-body cancelled-booking">
                <div class="card-text">
                    <h1>Cancelled</h1>
                    <hr />
                    <div v-if="cancelled.length === 0">No cancelled booking request has been cancelled.</div>
                    <div v-for="booking in cancelled" :key="booking.id">
                        {{booking.user.name}}'s request for {{ booking.quantity }} cop{{ booking.quantity > 1 ? 'ies' : 'y' }} of
                        <router-link :to="'/books/' + booking.book.id">{{ booking.book.title }}</router-link>  has been cancelled
                        <hr />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data () {
        return {
            pending: [],
            approved: [],
            cancelled: []
        };
    },

    mounted () {
        this.fetchBookings();
    },

    methods: {
        async fetchBookings () {
            let bookings = await this.$http.get('/books/bookings');
            bookings = bookings.data.bookings;
            bookings.sort((a, b) => {
                return new Date(a.createdAt) - new Date(b.createdAt);
            });

            // Fetch informations about books
            let bookIDs = bookings.map((booking) => booking.book_id);
            bookIDs = [...new Set(bookIDs)];
            bookIDs = bookIDs.map((id) => this.$http.get(`/books/${id}`));

            // Fetch informations about users
            let userIDs = bookings.map((booking) => booking.user_id);
            userIDs = [...new Set(userIDs)];
            userIDs = userIDs.map((id) => this.$http.get(`/users/${id}`));

            // Extract informations about books
            let bookMap = {};
            for (let book of bookIDs) {
                book = await book;
                book = book.data.book;
                bookMap[book._id] = {
                    id: book._id,
                    title: book.title
                };
            }

            // Extract informations about users
            let userMap = {};
            for (let user of userIDs) {
                user = await user;
                user = user.data.user;
                userMap[user._id] = {
                    id: user._id,
                    name: user.name
                };
            }

            bookings = bookings.map((booking) => {
                return {
                    id: booking._id,
                    user: userMap[booking.user_id],
                    book: bookMap[booking.book_id],
                    quantity: booking.quantity,
                    status: booking.status
                };
            });

            this.pending = bookings.filter((booking) => {
                return booking.status === 'pending';
            });

            this.approved = bookings.filter((booking) => {
                return booking.status === 'approved';
            });

            this.cancelled = bookings.filter((booking) => {
                return booking.status === 'cancelled';
            });
        },

        approve (bookingID) {
            this.$http.patch(`/books/bookings/${bookingID}`, {
                status: 'approved'
            }).then(() => {
                this.fetchBookings();
            });
        },

        cancelBooking (bookingID) {
            this.$http.patch(`/books/bookings/${bookingID}`, {
                status: 'cancelled'
            }).then(() => {
                this.fetchBookings();
            });
        },

        update (bookingID, quantity) {
            this.$http.patch(`/books/bookings/${bookingID}`, {
                quantity
            }).then(() => {
                this.fetchBookings();
            });
        }
    }
};
</script>

<style lang="scss" scoped>
.btn {
    margin-right: 10px;
}
</style>
