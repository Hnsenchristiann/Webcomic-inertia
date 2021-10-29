<template>
    <app-layout>
        <div class="rounded-xl m-5 bg-gray-200 bg-opacity-70 border border-indigo-600">
            <div class="flex flex-row p-5">
                <div>
                    <img class="w-24 h-36" :src="author.profile_picture_url" />
                </div>
                <div>
                    <div class="ml-4 text-2xl font-bold">
                        {{author.name}}
                    </div>
                    <div class="ml-4 text-lg mt-3">
                        {{author.email}}
                    </div>
                    <div>
                        <div class="ml-4 mt-3 text-sm">
                            Social Media :
                        </div>
                        <div class="flex items-start justify-start ml-2 mt-2">
                            <a v-for="(soc, idx) in socials" :key="'soc-' + idx" :href="soc.link">
                                <img class="w-8" :src="soc.icon" />
                            </a>
                        </div>
                    </div>
                </div>
            </div>
            <div class="p-5">
                <div class="text-xl font-bold">About Author :</div>
                <div>{{author.description}}</div>
            </div>
            <div class="p-5">
                <div class="text-xl font-bold">Author Books :</div>
                <div class="mb-3">
                    <div>
                        <horizontal-slider :items="processToHorizontalSlider(comics.all.comics)"
                                        :config="config"
                                        objectCategory="all"
                                        @nextPage="nextPage"></horizontal-slider>
                    </div>
                </div>
            </div>
        </div>
    </app-layout>
</template>

<script>
    import HorizontalSlider from '../Components/HorizontalSlider.vue'
    import AppLayout from '@/Layouts/AppLayout.vue'
    export default {
        name: 'author',
        components: {
            HorizontalSlider,
            AppLayout
        },
        props: ['author'],
        data() {
            return {
                icons: {
                    facebook: require('../../icons/facebook.png'),
                    instagram: require('../../icons/instagram.png'),
                    twitter: require('../../icons/twitter.png'),
                    visi8: require('../../assets/visi8_logo.png')
                },
                // author: [],
                shownTags: [
                    'asdf',
                    'lorem'
                ],
                comics: {
                    all: {
                        comics: []
                    }
                },
                query: {
                    paginate: 5,
                    page: 1
                },
                config: {
                    image: 'cover_url',
                    title: 'title'
                },
                socials: [],
            }
        },
        created() {
            this.shownTags.forEach((elem) => {
                this.comics[elem] = {}
                this.comics[elem].comics = []
                this.getComics(route('api.comics.list', { ...this.query, where_tag: elem }), elem)
            }),
            axios.get(route('api.settings', 'site.social_media_links'))
            .then((resp) => {
                this.socials = Object.keys(resp.data).map((socname) => {
                    const icon = socname in this.icons ? this.icons[socname] : this.icons.visi8
                    return {
                        name: socname,
                        link: resp.data[socname],
                        icon: icon.default
                    }
                })
            })
            this.getComics(route('api.comics.list', this.query), 'all')
            // axios.get(route('api.author.show', { author: this.$route.params.authorId }))
            //     .then((response) => {
            //         this.authors = response.data
            //     })
        },
        methods: {
            processToHorizontalSlider(comicObjects) {
                let retVal = []
                comicObjects.forEach(element => {
                    retVal.push({
                        url: '/comic/' + element.id,
                        cover_url: element.cover_url,
                        title: element.title
                    })
                });
                return retVal
            },
            getComics(url, category) {
                axios.get(url)
                    .then((response) => {
                        if (!this.comics[category]) {
                            this.comics[category] = {}
                            this.comics[category].comics = response.data.data
                        } else {
                            this.comics[category].comics = this.comics[category].comics.concat(response.data.data)
                        }
                        this.comics[category].paginationData = response.data
                        this.comics[category].prevDisabled = this.comics[category].paginationData.prev_page_url === null
                        this.comics[category].nextDisabled = this.comics[category].paginationData.next_page_url === null
                        this.comics[category].prevPageUrl = this.comics[category].paginationData.prev_page_url
                        this.comics[category].nextPageUrl = this.comics[category].paginationData.next_page_url
                    })
                    .catch((error) => {
                        //do error catching later
                    })
            },
            nextPage(category) {
                if (!this.comics[category].nextDisabled) {
                    this.getComics(this.comics[category].nextPageUrl, category)
                }
            },
            prevPage(category) {
            }
        }
    }
</script>

<style>
</style>
