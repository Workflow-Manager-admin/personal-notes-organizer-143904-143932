<script lang="ts">
	import Sidebar from '$lib/components/Sidebar.svelte';
	import Topbar from '$lib/components/Topbar.svelte';
	import NotesEditor from '$lib/components/NotesEditor.svelte';
	import SettingsModal from '$lib/components/SettingsModal.svelte';
	import Auth from '$lib/components/Auth.svelte';

	import { authToken, notes, selectedNoteId } from '$lib/store';
	import { getNotes } from '$lib/api';

	let initialized = false;
	let loading = false;
	let newNote = false;

	// Avoid infinite-loop: Only fetch notes when not initialized & token exists
	$: if ($authToken && !initialized) {
		loading = true;
		getNotes($authToken)
			.then((data) => {
				notes.set(data || []);
			})
			.finally(() => {
				initialized = true;
				loading = false;
			});
	}
	// Only reset if initialized was true
	$: if (!($authToken) && initialized) {
		notes.set([]);
		selectedNoteId.set(null);
		initialized = false;
	}

	function handleSidebarAdd() {
		newNote = true;
		selectedNoteId.set(null);
	}
	function handleSidebarSelect({ detail }) {
		newNote = false;
		selectedNoteId.set(detail.id);
	}
	function handleLogout() {
		initialized = false;
	}
</script>

<svelte:head>
	<title>Personal Notes App</title>
	<meta name="description" content="Minimalist, modern notes app" />
</svelte:head>

{#if !$authToken}
	<Auth />
{:else}
	<div class="app-frame">
		<Sidebar on:add={handleSidebarAdd} on:select={handleSidebarSelect} />
		<main class="main-area">
			<Topbar on:logout={handleLogout} />
			{#if loading}
				<div class="loading">Loading notes…</div>
			{:else}
				<NotesEditor
					note={$notes.find((n) => n.id === $selectedNoteId)}
					isNew={newNote}
				/>
			{/if}
		</main>
		<SettingsModal />
	</div>
{/if}

<style>
.app-frame {
	display: flex; min-height: 100vh; background: #f5f7fa;
}
.main-area {
	flex: 1 1 auto; display: flex; flex-direction: column;
	padding: 0; background: #f5f7fa;
}
.loading {
	margin: 60px auto; text-align: center; color: #1976d2; font-size: 1.3rem;
}
@media (max-width: 700px) {
	.app-frame { flex-direction: column; }
	.main-area { padding: 0; }
}
</style>
