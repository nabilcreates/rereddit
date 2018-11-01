<template>
    <Page>
        <ActionBar :title="app.title" />
        <StackLayout>


            <Label textWrap="true" style="margin-bottom: 30; font-weight: 700; color: red;">THIS APP IS UNRELEASED AND
                IS IN ITS VERY VERY BETA STAGE, MEANING THAT AT THIS POINT OF TIME, BUGS ARE CONSIDERED NORMAL</Label>


            <!-- UI CHANGES, ADDED APP TITLE AND ALSO THE DESCRIPTION -->
            <Label textWrap="true" style=" font-size: 30; font-weight: 700;">{{app.title}}</Label>
            <Label textWrap="true"> {{app.description}} </Label>
            <Label textWrap="true" style="margin-bottom: 30;"> App Current Status: {{app.status}} </Label>

            <!-- CHANGE SUBREDDIT INPUTFIELD -->
            <TextField v-model="subreddit" />

            <!-- REFRESH BUTTON -->
            <Button @tap="refresh()"> Refresh / Update </Button>

            <!-- LISTVIEW / LOOP -->
            <ListView class="list-group" for="post in redditdata.data.children" @itemTap="onPostTap" style="height:1250px"
                v-if="loaded">
                <v-template>
                    <FlexboxLayout flexDirection="row" class="list-group-item">
                        <Label :text="post.data.title" class="list-group-item-heading" />

                        <Img :src="post.data.thumbnail" style="width: 200px;" />

                    </FlexboxLayout>
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
                    status: "launched",
                },

                subreddit: "wallpapers",

                redditdata: [],

                loaded: false,

            }

        },

        methods: {

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
                this.verifySubreddit()
            },

            verifySubreddit() {

                fetch("https://www.reddit.com/subreddits/search.json?q=" + this.subreddit)
                    .then(res => res.json())
                    .then(json => {

                        console.log(json)

                        // CHECK IF THE SUBREDDIT EXISTS
                        if (json.data.children.length == 0) {

                            // IF IT DOESNT EXIST, APP.STATUS WILL PRINT OUT BELOW
                            this.app.status = "Subreddit doesn't exist"

                        } else {

                            // IF IT EXISTS
                            this.fetchApiData()

                        }

                    })
            },

            fetchApiData() {

                fetch("https://www.reddit.com/r/" + this.subreddit + "/new.json?limit=100")
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

        },

        mounted() {
            this.verifySubreddit()
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
        margin: 30;
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
    }

    Img {
        border-radius: 100;
    }
</style>