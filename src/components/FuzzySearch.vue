
<template>

    <div class="fuzzySearch">
        <h2 class="fuzzySearchStateLabel">
            <div class="decoration"></div>
            Start typing to search products
        </h2>
        <div class=""></div>
        <div class="icon">
            <img src="../assets/images/search.svg" alt="">
        </div>

        <div class="resultsList">

            <div class="label">Nothing matched your search</div>

            <div class="result" @click="this.viewProduct(product)" v-for="(product) in searchResults">

                <div class="left">
                    <!-- <div style="background:white url('https://i.dummyjson.com/data/products/7/3.jpg');background-size: cover;"
                        class="thumbnail">
                    </div> -->
                    <div class="thumbnail" :style='{ backgroundImage: `url(${product["images"].pop()})` }'>
                    </div>

                    <div class="name">

                        <div class="title">{{ product['title'] }}</div>
                        <div class="brand">{{ product['brand'] }}</div>
                    </div>
                </div>
                <div class="right">
                    <div class="price">${{ product['price'] }}</div>
                    <div class="rating">
                        <div class="star" v-for="n in Math.round(product['rating'])"> <img
                                src="../assets/images/ratingStar.svg" /></div>
                    </div>
                </div>

            </div>
        </div>

        <input v-model="searchQuery" placeholder="Search products..." type="text" id="productFuzzySearchInput">
        <div class="microloader">
            <div></div>
        </div>

        <div class="selectedProductIndicator">
            <div class="selectedProductIndicator">
                <div class="brand">{{ selectedProductBrand }}</div>
                <div class="title">{{ selectedProductTitle }}</div>

            </div>
        </div>

        <div @click="clearSearchQuery()" class="clear">
            <img src="../assets/images/clearSearchIcon.svg" />
        </div>
    </div>

</template>

<script lang="ts">

import { onMounted, defineComponent, watch } from "vue"
import { gsap, Linear, Expo } from 'gsap'


export default defineComponent({
    props: {
        searchQuery: {
            type: String
        },
    },
    methods: {
        queryProducts(queryString) {
            console.log(`Ready to search for ${queryString}`)
            fetch(`https://dummyjson.com/products/search?q=${queryString}`)
                .then(response => response.json())
                .then(data => this.parseSearchResults(data))
                .catch(error => {
                    console.log('Error querying products:', error);
                    this.hideMicroloader()
                })
        },

        viewProduct(product) {

            this.selectedProductBrand = product.brand
            this.selectedProductTitle = product.title
            this.showClearButton();
            gsap.to(`.resultsList`, {
                ease: Expo.easeOut,
                x: -13,
                autoAlpha: 0,
                duration: 0.55
            })
        }
        ,
        parseSearchResults(results) {
            console.log(`Received search results as ${typeof (results)}`)
            results = results.products;
            this.searchResults = results;
            this.searchResultsCount = results.length;

            console.log(results)
            console.log(`Returned ${this.searchResultsCount} results`)
            this.showSearchResults()
            results.forEach(result => {

                console.log(`Received result as ${result.title}`);
                let resultThumbail = result.images.pop();
                console.log(`Got thumbnail link as ${resultThumbail}`);
                // document.querySelector(`.resultsList`)?.insertAdjacentHTML(`afterbegin`, `
                // <div onclick="clearSearchQuery()" class="result">
                // <div style="font-family:'title'">${result.title}</div>   
                // <div v-html="${result.brand}"" class="brand">${result.brand}</div>    
                // <div class="brand">${result.rating}</div>    
                // <div class="brand">$${result.price}</div>    
                // </div>
                // `)

            });
        }
        ,
        showClearButton() {
            console.log(`Showing the clear search query button`)
            gsap.to((`.clear`), {
                autoAlpha: 1,
                scale: 1,
            })
        },
        showSearchResults() {
            console.log(`Showing search results...`);
            gsap.to(`.resultsList`, {
                autoAlpha: 1
            })
        }
        ,
        hideSearchResults() {
            console.log(`Hiding search results...`)
            gsap.to(`.resultsList`, {

                autoAlpha: 0
            })
        }
        ,
        hideClearButton() {
            console.log(`Hiding the clear search query button`)
            gsap.to(`.clear`, {
                autoAlpha: 0,
                scale: 0.89,
            })
        },
        clearSearchQuery() {
            console.log(`Clearing search query...`)
            // this.searchQuery = ``
            gsap.to(`.microloader`, {
                autoAlpha: 0
            })
            gsap.to(`.clear`, {
                autoAlpha: 0
            })
            gsap.to(`.resultsList`, {
                autoAlpha: 1,
                x: 0
            })
            { { this.selectedProductBrand = '' } }
            { { this.selectedProductTitle = '' } }
        },
        showMicroloader() {
            console.log(`Showing microloader`);

            gsap.to(`.microloader`, {
                rotation: `+=360`,
                repeat: -1,
                ease: Linear.easeNone
            })
            gsap.to(`.microloader`, {
                autoAlpha: 1,
            })
        },
        hideMicroloader() {
            console.log(`Hiding microloader`)
            gsap.to(`.microloader`, {
                autoAlpha: 0,
                onComplete: function () {
                    gsap.killTweensOf('.microloader')
                }
            })
        },
        countSearchResults() {
            console.log(`Counting search results...`)
        }
    },
    data() {
        return {
            searchQuery: '',
            searchResponse: '',
            searchResults: [],
            searchResultsCount: 0,
            selectedProductTitle: '',
            selectedProductBrand: ''
        }
    },

    watch: {
        // Whenever the fuzzySearch input changes, this will run
        searchQuery(newQuery) {
            this.showMicroloader()
            gsap.to(`.clear`, {
                duration: 0.08,
                scale: 0.89,
                autoAlpha: 0
            })
            setTimeout(() => {
                if (newQuery.length > 0) {
                    this.queryProducts(newQuery)
                    this.showSearchResults()
                }

                //Hides the clear search query button if the user removes all characters from the search

            }, 1555);
            if (newQuery.length == 0) {
                this.hideClearButton()
                this.hideMicroloader()
                this.hideSearchResults()
            }
        },
        searchResultsCount() {

            if (this.searchResultsCount > 0) {
                gsap.set(`.resultsList .label`, { autoAlpha: 0, fontSize: '0em' });

                this.hideMicroloader()
            }
            else {
                gsap.set(`.resultsList .label`, { autoAlpha: 1, fontSize: `initial` })
                this.hideMicroloader()
            }

        }
    },
    setup() {


        onMounted(async () => {
            console.log(`Now DOM elements are ready...`);
        })
    }
})

</script>

<style scoped>
.fuzzySearch:first-of-type {
    margin-top: 0px
}

.fuzzySearch {
    width: 100%;
    max-width: 384px;
    position: relative;
    margin-top: 34px;
}

.fuzzySearchStateLabel {
    position: relative;
    bottom: 13px;
    width: fit-content;
    color: #32274D
}

.fuzzySearchStateLabel .decoration {
    position: absolute;
    width: 64px;
    height: 5px;
    background: #6366F1;
    top: 30px;
    border-radius: 5px;
}

.fuzzySearch .icon {
    position: absolute;
    bottom: 5px;
    left: 10px;
    z-index: 1;
}

.fuzzySearch .icon img {
    width: 20px
}

.fuzzySearch input {
    box-shadow: 0px 0px 13px rgba(0, 0, 0, 0.21);
    border: 0px;
    border-radius: 21px;
    padding: 10px 13px 10px 34px;
    width: 100%;
    font-size: 1.05em;
    transition: 0.21s all cubic-bezier(.17, .67, .83, .67);
}

.fuzzySearch input:focus {
    outline: none;
    box-shadow: 0px 0px 13px rgba(99, 102, 241, 0.89);
}

.fuzzySearch .clear {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    right: 12px;
    bottom: 10px;
    border-radius: 13px;
    width: 21px;
    height: 21px;
    background: white;
    z-index: 2;
    cursor: pointer;
    transform: scale(0.89);
    visibility: hidden;
}

.fuzzySearch .microloader {
    position: absolute;
    right: 15px;
    bottom: 13px;
    visibility: hidden;
}

.fuzzySearch .microloader div {
    width: 16px;
    height: 16px;
    border-left: solid 1px #6366f0;
    border-top: solid 1px #6366f0;
    border-right: solid 1px #6366f0;
    border-radius: 13px;
}

.fuzzySearch .resultsList {
    position: absolute;
    margin-top: 50px;
    width: 100%;
    visibility: hidden;
}

.fuzzySearch .resultsList .result:first-child {
    margin: 0px 0px 0px 0px;
}

.fuzzySearch .resultsList .label {
    border-radius: 13px;
    background: white;
    box-shadow: 0px 1px 13px rgb(0 0 0 / 21%);
    padding: 8px;
    margin: 8px 0px 0px 0px;
    position: absolute;
}

.fuzzySearch .resultsList .result {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    cursor: pointer;
    border-radius: 13px;
    background: white;
    box-shadow: 0px 1px 13px rgb(0 0 0 / 21%);
    padding: 8px;
    margin: 8px 0px 0px 0px;
    transition: 0.13s ease-in-out all;
}

.fuzzySearch .resultsList .result:hover {
    transform: scale(1.01);
}

.fuzzySearch .resultsList .result .left,
.fuzzySearch .resultsList .result .right {
    display: flex;

}

.fuzzySearch .resultsList .result .left {}

.fuzzySearch .resultsList .result .left .name {
    padding: 0px 0px 0px 5px;
}

.fuzzySearch .resultsList .result .right {
    text-align: right;
    flex-direction: column;
}

.fuzzySearch .resultsList .result .title {
    font-family: "title";
    line-height: 16px;
    color: #32274d;
}

.fuzzySearch .resultsList .result .thumbnail {
    border-radius: 34px;
    box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.21);
    background-size: contain;
    background-position: center;
    background-repeat: no-repeat;
    width: 34px;
    height: 34px;
}

.fuzzySearch .selectedProductIndicator {
    position: absolute;
    bottom: 5px;
    left: 0px;
    width: 100%;
    background: white;
    display: flex;
    z-index: 1;
    padding-left: 17px;
    border-radius: 13px;
}

.fuzzySearch .selectedProductIndicator .brand {
    color: rgba(0, 0, 0, 0.64);
    padding: 0px 5px 0px 0px;
}

.fuzzySearch .selectedProductIndicator .title {
    font-family: 'title';
    color: #32274d;
}

.fuzzySearch .resultsList .result .brand {
    font-size: 0.55em;
    color: rgba(0, 0, 0, 0.64);
    padding: 3px 0px 0px 0px;
}

.fuzzySearch .resultsList .result .rating {
    display: flex;
    top: -10px;
    flex-direction: row;
    justify-content: flex-end;

}

.fuzzySearch .resultsList .result .rating .star {}

.fuzzySearch .resultsList .result .rating .star img {
    width: 8px;
}
</style>