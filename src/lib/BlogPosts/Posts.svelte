<script>
    import { goto } from "$app/navigation";
    import Pagination from "$lib/Pagination.svelte";
    import { getBlogPosts, leagueName } from "$lib/utils/helper";
    import LinearProgress from "@smui/linear-progress/LinearProgress.svelte";
    import { onMount } from "svelte";
    import Post from "./Post.svelte";

    export let postsData, queryPage = 1, filterKey = '';

    let page = queryPage - 1;

    let loading = true;
    let allPosts = [];
    let posts = [];

    let categories;

    const filterPosts = (ap, fk) => {
        if(ap.length && fk != '') {
            posts = ap.filter(p => p.fields.type == fk);
        } else {
            posts = ap;
        }
    }

    $: filterPosts(allPosts, filterKey);

    onMount(async ()=> {
        const startPostData = await postsData;
        allPosts = startPostData.posts;
        loading = false;

        const categoryMap = new Set();
        for(const post of startPostData.posts) {
            categoryMap.add(post.fields.type);
        }
        categories = [...categoryMap];

        if(!startPostData.fresh) {
            const blogResponse = await getBlogPosts(true);
            allPosts = blogResponse.posts;
            const categoryMap = new Set();
            for(const post of blogResponse.posts) {
                categoryMap.add(post.fields.type);
            }
            categories = [...categoryMap];
        }
    })

    const perPage = 10;
    $: total = posts.length;

    let el;
    $: top = el?.getBoundingClientRect() ? el?.getBoundingClientRect().bottom  : 0

    $: displayPosts = posts.slice(page * perPage, (page + 1) * perPage);

    let direction = 1;

    const changePage = (dest) => {
        if(dest + 1 > queryPage) {
            direction = 1;
        } else {
            direction = -1;
        }
        setTimeout(() => {goto(`/blog?page=${dest + 1}&filter=${filterKey}`, {noscroll: true,  keepfocus: true})}, 800);
    }

	$: changePage(page);
</script>

<style>
    h2 {
        font-size: 3em;
        text-align: center;
        margin-bottom: 0.2em;
    }
	.loading {
		display: block;
		position: relative;
		z-index: 1;
		width: 85%;
		max-width: 500px;
		margin: 80px auto;
	}
    .filter {
        display: inline-flex;
        color: #fff;
        border-radius: 2em;
        font-size: 0.8em;
        padding: 0.25em 1em;
    }

    .noUnderline {
        margin: 0.5em;
        text-decoration: none;
    }

    .filterClear {
        background-color: #920505;
    }

    .filterClear:hover {
        background-color: #720404;
    }

    .filterLink {
        background-color: #00316b;
    }

    .filterLink:not(.noHover):hover {
        background-color: #0082c3;
    }

    .noHover {
        cursor: default;
    }

    .filterButtons {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        margin: 1em 0 3em;
    }

    .filteringBy {
        font-size: 1em;
    }
</style>

<h2 bind:this={el}>{leagueName} Blog</h2>

{#if loading}
    <div class="loading" >
        <p>Loading league transactions...</p>
        <LinearProgress indeterminate />
    </div>
{:else}
    <div class="filterButtons">
        {#if filterKey == ''}
            {#each categories as category}
                <a class="noUnderline" href="/blog?filter={category}&page=1"><div class="filter filterLink">{category}</div></a>
            {/each}
        {:else}
            <div class="filteringBy">Showing <div class="filter filterLink noHover">{filterKey}</div> posts <a class="noUnderline" href="/blog?filter=&page=1"><div class="filter filterClear">Clear Filter</div></a></div>
        {/if}
    </div>

    <Pagination {perPage} {total} bind:page={page} target={top} scroll={false} />

    {#each displayPosts as post}
        <Post createdAt={post.sys.createdAt} post={post.fields} id={post.sys.id} {direction} />
    {/each}
    <Pagination {perPage} {total} bind:page={page} target={top} scroll={true} />
{/if}
