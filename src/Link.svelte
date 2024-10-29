<script>
    import { createEventDispatcher, getContext } from "svelte";
    import { HISTORY, LOCATION, ROUTER } from "./contexts.js";
    import { resolve, shouldNavigate } from "./utils.js";

    /**
     * @typedef {Object} Props
     * @property {string} [to]
     * @property {boolean} [replace]
     * @property {any} [state]
     * @property {any} [getProps]
     * @property {boolean} [preserveScroll]
     * @property {import('svelte').Snippet<[any]>} [children]
     */

    /** @type {Props & { [key: string]: any }} */
    let {
        to = "#",
        replace = false,
        state = {},
        getProps = () => ({}),
        preserveScroll = false,
        children,
        ...rest
    } = $props();

    const location = getContext(LOCATION);
    const { base } = getContext(ROUTER);
    const { navigate } = getContext(HISTORY);
    const dispatch = createEventDispatcher();

    let href = $derived(resolve(to, $base.uri)),
        isPartiallyCurrent = $derived($location.pathname.startsWith(href)),
        isCurrent = $derived(href === $location.pathname),
        props = $derived(
            getProps({
                location: $location,
                href,
                isPartiallyCurrent,
                isCurrent,
                existingProps: rest,
            })
        );

    let ariaCurrent = $derived(isCurrent ? "page" : undefined);

    const onClick = (event) => {
        dispatch("click", event);
        if (shouldNavigate(event)) {
            event.preventDefault();
            // Don't push another entry to the history stack when the user
            // clicks on a Link to the page they are currently on.
            const shouldReplace = $location.pathname === href || replace;
            navigate(href, { state, replace: shouldReplace, preserveScroll });
        }
    };
</script>

<a {href} aria-current={ariaCurrent} onclick={onClick} {...props} {...rest}>
    {@render children?.({ active: !!ariaCurrent })}
</a>
