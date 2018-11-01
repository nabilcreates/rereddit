<template>
    <Page>
        <ActionBar :title="app.title" />
        <StackLayout>

            <!-- MAIN CONCEPT OF THE API FETCHING IS THAT THE SUBREDDIT WILL BE VERIFIED BY VERIFYSUBREDDIT AND IF IT PASS, IT WILL RUN THROUGH FETCHAPI WHICH WILL GET THE POSTS -->

            <!-- UI CHANGES, ADDED APP TITLE AND ALSO THE DESCRIPTION -->
            <!-- <Label textWrap="true" style=" font-size: 30; font-weight: 700;">{{app.title}}</Label>
            <Label textWrap="true"> {{app.description}} </Label> -->
            <Label textWrap="true">Status: {{app.status}} </Label>
            <Label textWrap="true" style="margin-bottom: 10;">Mode: {{app.mode}} </Label>

            <!--SUBREDDIT INPUTFIELD -->
            <StackLayout class="input-field">
                <Label textWrap='true' text="Enter a subreddit:" />
                <TextField v-model="subreddit" />
            </StackLayout>

            <!-- REFRESH BUTTON -->
            <Button @tap="refresh()"> Refresh / Update </Button>

            <!-- SEARCH BUTTON -->
            <Button @tap="searchSubreddit()">Seach For Subreddit </Button>

            <!-- LISTVIEW / LOOP FOR SEARCH-->
            <!-- MAIN CONCEPT IS THAT IT WILL SEARCH AND THEN THE LISTVIEW WILL SHOW IF IT MEETS BOTH REQUIREMENT THAT IS LOADEDSEARC AND APP.MODE IS SEARCH. -->
            <ListView class="list-group" for="result in searchdata.data.children" @itemTap="onSearchTap" v-if="loadedsearch && app.mode == 'search'"
                separatorColor="gray">
                <v-template>
                    <StackLayout flexDirection="row" class="list-group-item">
                        <Label textWrap='true' :text="result.data.display_name_prefixed" class="list-group-item-heading maincolor" />
                        <Label textWrap='true' v-if="result.data.public_description" :text="result.data.public_description" class="list-group-item-heading" />
                        <Label textWrap='true' v-else text="No Description Provided" class="list-group-item-heading" />
                    </StackLayout>
                </v-template>
            </ListView>

            <!-- LISTVIEW / LOOP FOR SUBREDDITS-->
            <ListView class="list-group" for="post in redditdata.data.children" @itemTap="onPostTap" v-if="loaded && app.mode == 'subreddit'"
                separatorColor="gray">
                <v-template>
                    <StackLayout flexDirection="row" class="list-group-item">
                        <Img :src="post.data.thumbnail" style="width: 250px;" />
                        <Label textWrap='true' :text="post.data.title" class="list-group-item-heading" />
                    </StackLayout>
                </v-template>
            </ListView>


        </StackLayout>
    </Page>
</template>

<script>
    var utilsModule = require("tns-core-modules/utils/utils");

    export default {
        data() {
            return {

                app: {
                    title: "Rereddit",
                    description: "A Reddit Client for Android (BETA)",
                    status: "App Launched",
                    mode: "?",
                },

                subreddit: "wallpapers",

                redditdata: [],
                searchdata: [],

                loaded: false,
                loadedsearch: false,


            }

        },

        methods: {

            onSearchTap(args) {
                var subredditname = this.searchdata.data.children[args.index].data.display_name;
                console.log(subredditname);
                this.verifySubreddit(subredditname.toLowerCase())
            },

            onPostTap(args) {
                console.log(args.index)


                // GETTING PERMA LINK (IT RETURNS AS /R/SUBREDDIT SO WE NEED TO CONCAT IT TO THE REDDITLINK)
                var permalink = this.redditdata.data.children[args.index].data.permalink
                var redditlink = "https://reddit.com"

                // FULL LINK
                var link = redditlink + permalink
                console.log(link)
                utilsModule.openUrl(link)
            },

            refresh() {
                this.app.status = "Refreshing",
                    this.loaded = false
                this.verifySubreddit(this.subreddit)
            },

            searchSubreddit() {

                this.app.mode = "search";

                this.loadedsearch = false;
                this.app.status = "Searching for Subreddit"

                fetch("https://www.reddit.com/subreddits/search.json?q=" + this.subreddit)
                    .then(res => res.json())
                    .then(json => {

                        // json.data.children[i].data.display_name_prefixed
                        console.log(json)
                        this.searchdata = json

                        this.app.status = "Searched Subreddit"
                        this.loadedsearch = true;
                        this.loaded = false;
                    })


            },

            verifySubreddit(subreddit) {

                this.app.status = "Verifying Subreddit"
                this.app.mode = "verification"

                fetch("https://www.reddit.com/subreddits/search.json?q=" + subreddit)
                    .then(res => res.json())
                    .then(json => {

                        this.app.mode = "subreddit";
                        this.loadedsearch = false;

                        console.log(json)

                        // CHECK IF THE SUBREDDIT EXISTS
                        if (json.data.children.length == 0) {

                            // IF IT DOESNT EXIST, APP.STATUS WILL PRINT OUT BELOW
                            this.app.status = "Subreddit doesn't exist"

                        } else {

                            this.app.status = "Subreddit is valid. Fetching API..."

                            // IF IT EXISTS
                            fetch("https://www.reddit.com/r/" + subreddit + "/new.json?limit=100")
                                .then(res => res.json())
                                .then(json => {

                                    console.log(json)

                                    // ALL THE MAIN DATA STARTS AT json.data.children
                                    this.redditdata = json

                                    // ALL THE POST DATA START FROM json.data.children[x].data where x is the post number

                                    this.loaded = true;
                                    this.app.status = "Fetched API"
                                })

                        }

                    })
            },

        },

        mounted(){

            // THIS.SUBREDDIT EQUALS TO A RANDOM SUB
            var randsub = ["sbubby","memes","wallpaper","iwallpaper","apphookup","fallout","trees","natureislit","blackpeoplegifs","shittyreactiongifs","PhotoshopBattles","perfecttiming"]
            this.subreddit = randsub[(Math.floor(Math.random() * randsub.length))].toLowerCase()
        }
        

    }
</script>

<style scoped>
    /* PRIMARY COLOR IS #53b3ba */

    Page {
        background-color: #161616;
        color: #ffffff;
    }

    StackLayout {
        margin: 10;
    }

    ActionBar {
        background-color: #53b3ba;
        color: #ffffff;
    }

    .list-group-item-heading {
        color: #fff;
    }

    Button {
        background-color: #53b3ba;
        border-radius: 100;
        margin: 5 0;
    }

    .maincolor {
        color: #53b3ba;
        font-weight: 700;
    }
</style>