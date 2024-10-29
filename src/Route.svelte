<script lang="ts">
    import { getContext, onDestroy } from "svelte";
    import { ROUTER } from "./contexts.js";
    import { canUseDOM } from "./utils.js";

    let props = $props();
    let path = $state(props.path ?? "");
    let component = $state(props.component ?? null);

    let routeParams = $state({});
    let routeProps = $state({});

    const { registerRoute, unregisterRoute, activeRoute } = getContext(ROUTER);

    const route = {
        path,
        // If no path prop is given, this Route will act as the default Route
        // that is rendered if no other Route in the Router is a match.
        default: path === "",
    };

    $effect(() => {
        if ($activeRoute && $activeRoute.route === route) {
            routeParams = $activeRoute.params;

            const { component: c, path, ...rest } = props;
            routeProps = rest;

            if (c) {
                if (c.toString().startsWith("class ")) component = c;
                else component = c();
            }

            canUseDOM() &&
                !$activeRoute.preserveScroll &&
                window?.scrollTo(0, 0);
        }
    });

    registerRoute(route);

    onDestroy(() => {
        unregisterRoute(route);
    });
</script>

{#if $activeRoute && $activeRoute.route === route}
    {#if component}
        {#await component then resolvedComponent}
            {@render (resolvedComponent?.default || resolvedComponent)({
                ...routeParams,
                ...routeProps,
            })}
        {/await}
    {:else}
        {@render props.children?.({ params: { routeParams } })}
    {/if}
{/if}
