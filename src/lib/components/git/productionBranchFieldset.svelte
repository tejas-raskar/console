<script lang="ts">
    import { Button, InputText } from '$lib/elements/forms';
    import { Fieldset, Input, Layout, Selector, Skeleton } from '@appwrite.io/pink-svelte';
    import SelectRootModal from './selectRootModal.svelte';
    import { sdk } from '$lib/stores/sdk';
    import { sortBranches } from '$lib/stores/vcs';
    import { page } from '$app/state';

    export let branch = 'main';
    export let rootDir: string;
    export let silentMode: boolean;
    export let installationId: string;
    export let repositoryId: string;
    export let product: 'sites' | 'functions';

    let show = false;

    async function loadBranches() {
        const repo = await sdk
            .forProject(page.params.region, page.params.project)
            .vcs.getRepository(installationId, repositoryId);

        branch = repo.defaultBranch ?? 'main';

        const { branches } = await sdk
            .forProject(page.params.region, page.params.project)
            .vcs.listRepositoryBranches(installationId, repositoryId);

        return sortBranches(branches);
    }
</script>

<Fieldset legend="Branch">
    {#await loadBranches()}
        <Layout.Stack gap="xl">
            <Layout.Stack gap="xs">
                <Skeleton variant="line" width={100} height={20} />
                <Skeleton variant="line" width="100%" height={32} />
            </Layout.Stack>
            <Layout.Stack gap="xs">
                <Skeleton variant="line" width={100} height={20} />
                <Skeleton variant="line" width="100%" height={32} />
            </Layout.Stack>
            <Layout.Stack gap="xs">
                <Skeleton variant="line" width={100} height={20} />
                <Skeleton variant="line" width={300} height={15} />
                <Skeleton variant="line" width={300} height={15} />
            </Layout.Stack>
        </Layout.Stack>
    {:then branches}
        {@const options =
            branches
                ?.map((branch) => {
                    return {
                        value: branch.name,
                        label: branch.name
                    };
                })
                ?.sort((a, b) => {
                    return a.label > b.label ? 1 : -1;
                }) ?? []}
        <Layout.Stack gap="xl">
            <Input.ComboBox
                required
                id="branch"
                label="Production branch"
                placeholder="Select branch"
                bind:value={branch}
                on:select={(event) => {
                    branch = event.detail.value;
                }}
                {options} />
            <Layout.Stack direction="row" gap="s" alignItems="flex-end">
                <InputText
                    id="root"
                    label="Root directory"
                    placeholder="Select directory"
                    bind:value={rootDir} />
                <Button secondary size="s" on:click={() => (show = true)}>Select</Button>
            </Layout.Stack>

            <Selector.Checkbox
                size="s"
                id="silentMode"
                label="Silent mode"
                description="If selected, comments will not be created when pushing changes to this repository."
                bind:checked={silentMode} />
        </Layout.Stack>
    {/await}
</Fieldset>

{#if show}
    <SelectRootModal bind:show bind:rootDir {product} {branch} />
{/if}
