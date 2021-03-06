<template>
    <div class="media post">
        <vote :model="answer" name="answer"></vote>

        <div class="media-body">
            <form v-if="editing" @submit.prevent="update">
                <div class="form-group">
                    <textarea
                        v-model="body"
                        rows="10"
                        class="form-control"
                        required
                    ></textarea>
                </div>
                <button
                    class="btn btn-outline-primary"
                    type="submit"
                    :disabled="isInvalid"
                >
                    update
                </button>
                <button
                    class="btn btn-outline-secondary"
                    @click="cancel"
                    type="button"
                >
                    cancel
                </button>
            </form>
            <div v-else>
                <div v-html="bodyHtml"></div>
                <div class="row">
                    <div class="col-4">
                        <div class="ml-auto">
                            <a
                                v-if="authorize('modify', answer)"
                                @click.prevent="edit"
                                class="btn btn-sm btn-outline-info"
                                >Edit</a
                            >
                            <button
                                v-if="authorize('modify', answer)"
                                @click="destroy"
                                class="btn btn-sm btn-outline-danger"
                            >
                                Delete
                            </button>
                        </div>
                    </div>
                    <div class="col-4"></div>
                    <div class="col-4">
                        <user-info :model="answer" label="Answered"></user-info>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Vote from "./Vote.vue";
import UserInfo from "./UserInfo.vue";

export default {
    props: ["answer"],
    components: {
        Vote,
        UserInfo
    },
    data() {
        return {
            editing: false,
            body: this.answer.body,
            bodyHtml: this.answer.body_html,
            id: this.answer.id,
            questionID: this.answer.question_id,
            beforeEditCache: null
        };
    },
    computed: {
        isInvalid() {
            return this.body.length < 10;
        },
        endpoint() {
            return `/questions/${this.questionID}/answers/${this.id}`;
        }
    },
    methods: {
        edit() {
            this.beforeEditCache = this.body;
            this.editing = true;
        },
        cancel() {
            this.body = this.beforeEditCache;
            this.editing = false;
        },
        update() {
            axios
                .patch(this.endpoint, {
                    body: this.body
                })
                .then(res => {
                    this.editing = false;
                    this.bodyHtml = res.data.body_html;
                    this.$toast.success(res.data.message, "success", {
                        timeout: 3000
                    });
                })
                .catch(err => {
                    this.$toast.error(err.response.data.message, "error", {
                        timeout: 3000
                    });
                });
        },
        destroy() {
            // izitoast confirm
            this.$toast.question("Are you sure about that?", "confirm", {
                timeout: 20000,
                close: false,
                overlay: true,
                displayMode: "once",
                id: "question",
                zindex: 999,
                title: "Hey",
                position: "center",
                buttons: [
                    [
                        "<button><b>YES</b></button>",
                        (instance, toast) => {
                            // delete async request
                            axios.delete(this.endpoint).then(res => {
                                this.$emit("deleted");
                            });
                            // delete async request
                            instance.hide(
                                { transitionOut: "fadeOut" },
                                toast,
                                "button"
                            );
                        },
                        true
                    ],
                    [
                        "<button>NO</button>",
                        function(instance, toast) {
                            instance.hide(
                                { transitionOut: "fadeOut" },
                                toast,
                                "button"
                            );
                        }
                    ]
                ]
            });
        }
    }
};
</script>
