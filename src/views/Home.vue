<template>
<div class="container home">
	<div class="header">
		Github Search
	</div>

	<div class="result-container">
		<div class="control search-input">
			<input class="input is-small" type="text" v-model="searchStr" @keyup.enter="runApi(1)" placeholder="Search">
		</div>
		<div class="control result-label">
			<div v-if="searchStr == null || searchStr == ''">
				Please enter keyword and press 'Enter'
			</div>
			<div v-if="results.length">
				{{ this.totalRepo }} repository results
			</div>
			<div v-if="isSearch && results.length == 0">
				No repository was found
			</div>
		</div>
		<div class="underline"></div>
		<div class="columns" v-for="(result, index) in results" :key="'result' + index">
			<div class="column is-two-thirds outcome-label">
				<a :href="result.html_url" target="_blank">{{result.full_name}}</a>
				<div class="outcome-desc">
					{{result.description}}
				</div>
				<div class="outcome-desc lighter">
					Updated on {{new Date(result.updated_at).toDateString()}}
				</div>
			</div>
			<div class="column outcome-lang">
				<fa-icon icon="circle" size="xs"/> {{result.language}}
			</div>
			<div class="column outcome-star">
				<fa-icon icon="star" size="xs"/> {{result.stargazers_count}}
			</div>
		</div>
		<div v-if="results.length != 0" class="pagination-btn">
			<a class="button page-btn" @click="backward"><fa-icon icon="chevron-left"/></a>
			<a class="button page-btn" @click="pageClick(pageArray[0])" :class="{'active': currentPage == pageArray[0]}">{{pageArray[0]}}</a>
			<a class="button page-btn" @click="pageClick(pageArray[1])" :class="{'active': currentPage == pageArray[1]}">{{pageArray[1]}}</a>
			<a class="button page-btn" @click="pageClick(pageArray[2])" :class="{'active': currentPage == pageArray[2]}">{{pageArray[2]}}</a>
			<a class="button page-btn" @click="pageClick(pageArray[3])" :class="{'active': currentPage == pageArray[3]}">{{pageArray[3]}}</a>
			<span class="button page-btn ellipse">&centerdot;&centerdot;&centerdot;</span>
			<a class="button page-btn" @click="lastCount">{{totalCount}}</a>
			<a class="button page-btn" @click="forward"><fa-icon icon="chevron-right"/></a>
		</div>
	</div>
</div>
</template>

<script>
import axios from 'axios'
export default {
	data() {
		return {
			searchStr: null,
			results: [],
			totalCount: null,
			totalRepo: null,
			pageArray: [],
			currentPage: null,
			isSearch: false
		}
	},
	methods: {
		runApi(page) {
			this.isSearch = false
			this.pageArray = [1, 2, 3, 4]
			this.currentPage = 1
			axios.get('https://api.github.com/search/repositories?page=' + page + '&per_page=10&q=' + this.searchStr).then(result => {
				this.totalRepo = result.data.total_count
				this.totalCount = Math.ceil(result.data.total_count/10)
				this.results = result.data.items
				if (result.data.items.length == 0) {
					this.isSearch = true
				}
			}, e => {
				this.isSearch = true
			})
		},
		pageClick(page) {
			this.currentPage = page
			axios.get('https://api.github.com/search/repositories?page=' + page + '&per_page=10&q=' + this.searchStr).then(result => {
				this.results = result.data.items
			}, e => {
				alert('Only the first 1000 search results are available.')
			})
		},
		forward() {
			if (this.currentPage + 1 > this.totalCount) return 
			this.currentPage += 1
			if (!this.pageArray.includes(this.currentPage)) {
				this.pageArray = [this.currentPage, this.currentPage + 1, this.currentPage + 2, this.currentPage + 3]
			}
			axios.get('https://api.github.com/search/repositories?page=' + this.currentPage + '&per_page=10&q=' + this.searchStr).then(result => {
				this.results = result.data.items
			}, e => {
				alert('Only the first 1000 search results are available.')
			})
		},
		backward() {
			if (this.currentPage - 1 == 0) return
			this.currentPage -= 1
			if (!this.pageArray.includes(this.currentPage)) {
				this.pageArray = [this.currentPage - 3, this.currentPage - 2, this.currentPage - 1, this.currentPage]
			}
			axios.get('https://api.github.com/search/repositories?page=' + this.currentPage + '&per_page=10&q=' + this.searchStr).then(result => {
				this.results = result.data.items
			}, e => {
				alert('Only the first 1000 search results are available.')
			})
		},
		lastCount() {
			axios.get('https://api.github.com/search/repositories?page=' + this.totalCount + '&per_page=10&q=' + this.searchStr).then(result => {
				this.currentPage = this.totalCount
				this.pageArray = [this.totalCount -4, this.totalCount - 3, this.totalCount - 2, this.totalCount - 1]
				this.results = result.data.items
			}, e => {
				alert('Only the first 1000 search results are available.')
			})
		}
	}
}
</script>

<style lang="scss">
.home {
	height: 100%;
	.columns {
		margin: inherit;
		border-bottom: 1px solid #EDEDED;
		.column {
			padding-left: 0;
			padding-right: 0;
		}
		&:last-child {
			border-bottom: 0;
		}
	}
	.underline {
		padding-top: 12px;
		border-bottom: 1px solid #EDEDED;
	}
	background-color: #F1F3F6;
	.header {
		padding: 6px 18px 28px;
	}
	.result-container {
		background-color: #FFFFFF;
		margin: 0 12px 12px 4px;
		padding: 12px;
		.result-label {
			font-weight: bold;
		}
		.search-input {
			padding-bottom: 24px;
		}
		.outcome-label {
			font-weight: bold;
			color: #5082F7;
			.outcome-desc {
				color: grey;
				font-size: 12px;
				opacity: .9;
				font-weight: 400;
				&.lighter {
					padding-top: 18px;
					opacity: .8;
					font-weight: 200;
				}
			}
		}
		.outcome-lang, .outcome-star {
			font-size:12px;
		}
		.pagination-btn {
			text-align: center;
			.page-btn {
				margin: 24px 2px;
				&.ellipse {
					border: 0;
				}
				&.active {
					background-color: #5082F7;
				}
			}
		}
	}
}

@media screen and (min-width: 1088px) {
	.container {
		max-width: 720px;
		width: 720px;
	}
}
</style>
