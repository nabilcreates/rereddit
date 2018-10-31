<template>
    <Page>
        <ActionBar :title="app.title" />
        <StackLayout>

            <Label textWrap="true" style="margin-bottom: 30;"> App Current Status: {{app.status}} </Label>

            <!-- REFRESH BUTTON -->
            <Button @tap="refresh()"> Refresh </Button>
            
            <!-- LISTVIEW / LOOP -->
            <ListView class="list-group" for="post in redditdata.data.children" style="height:1250px" v-if="loaded">
                <v-template>
                    <FlexboxLayout flexDirection="row" class="list-group-item">
                        <Label :text="post.data.title" class="list-group-item-heading" />

                        <Img :src="post.data.thumbnail" style="width: 50px;" />
                        
                    </FlexboxLayout>
                </v-template>
            </ListView>


        </StackLayout>
    </Page>
</template>

<script>
    export default {
        data() {
            return {

                app: {
                    title: "Rereddit",
                    status: "launched",
                },

                redditdata: [],

                loaded: false,

            }

        },

        methods: {

            refresh(){
                this.app.status = "Refreshing",
                this.loaded = false
                this.getApiData()
            },

            getApiData() {

                fetch("https://www.reddit.com/r/wallpapers/new.json")
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
            this.getApiData()
        }

    }
</script>

<style scoped>
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

    .list-group-item-heading{
        color: #fff;
    }

    Button{
        background-color: #53b3ba;
        border-radius: 100;
    }

    
</style>