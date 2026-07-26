<script lang="ts">
    import { onMount } from "svelte";
    import "../app.css";
    import Icon from "@iconify/svelte";
    import { env } from '$env/dynamic/public';

    import FoodCard from './FoodCard.svelte';
    import type { Meal } from '$lib/types';

    // Use environment variable for API URL with fallback to localhost
    let baseUrl = env.PUBLIC_API_URL || "http://localhost:8080";

    // For more information on runes and reactivity, see: https://svelte.dev/docs/svelte/what-are-runes
    let meals: Meal[] = $state([]);
    let loading = $state(true);
    let error = $state<string | null>(null);


    async function fetchMeals() {
        loading = true;
        error = null;
        try {
            const res = await fetch(`${baseUrl}/mensa-garching/today`);
            if (res.status === 204) {
                meals = [];
            } else if (res.ok) {
                meals = await res.json();
            } else {
                error = `Request failed with status ${res.status}`;
            }
        } catch (e) {
            error = e instanceof Error ? e.message : "Failed to load menu";
        } finally {
            loading = false;
        }
    }

    // Fetch data once on component mount
    onMount(async () => {
        await fetchMeals();
    });
</script>

<main>
    <header>
        <h1>Garching Campus Canteen</h1>
        <p>Today's menu offerings</p>
    </header>

    {#if loading}
        <div class="no-results">
            <p>Loading menu items...</p>
        </div>
    {:else if error}
        <div class="no-results">
            <p>Could not load the menu: {error}</p>
        </div>
    {:else if meals.length === 0}
        <div class="no-results">
            <p>No menu available today.</p>
        </div>
    {:else}
       <div class="food-grid">
            {#each meals as meal}
                <FoodCard {meal}/>
            {/each}
        </div>
    {/if}
</main>
