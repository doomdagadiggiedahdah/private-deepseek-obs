<script lang="ts">
  import { getContext } from "svelte"
  import type { Writable } from "svelte/store"

  let adminSection: Writable<string> = getContext("adminSection")
  adminSection.set("api-keys")

  let showCreateModal = $state(false)
  let newKeyName = $state("")
  let loading = $state(false)
  let createdKey = $state<string | null>(null)
  let showSuccess = $state(false)

  interface ApiKey {
    id: string
    name: string
    keyPrefix: string
    created: string
    status: "Active" | "Inactive"
  }

  let apiKeys = $state<ApiKey[]>([
    {
      id: "1",
      name: "Production API",
      keyPrefix: "api_abc...xyz123",
      created: "2024-01-15",
      status: "Active"
    },
    {
      id: "2", 
      name: "Development",
      keyPrefix: "api_def...456ghi",
      created: "2024-01-10",
      status: "Active"
    }
  ])

  function openCreateModal() {
    showCreateModal = true
    newKeyName = ""
    createdKey = null
    showSuccess = false
  }

  function closeModal() {
    showCreateModal = false
    newKeyName = ""
    createdKey = null
    showSuccess = false
  }

  async function createApiKey() {
    if (!newKeyName.trim()) return
    
    loading = true
    
    await new Promise(resolve => setTimeout(resolve, 1000))
    
    const newKey = {
      id: Date.now().toString(),
      name: newKeyName.trim(),
      keyPrefix: `api_${Math.random().toString(36).substring(2, 5)}...${Math.random().toString(36).substring(2, 8)}`,
      created: new Date().toISOString().split('T')[0],
      status: "Active" as const
    }
    
    apiKeys = [...apiKeys, newKey]
    createdKey = `api_${Math.random().toString(36).substring(2, 32)}`
    showSuccess = true
    loading = false
  }

  function copyToClipboard(text: string) {
    navigator.clipboard.writeText(text)
  }

  async function deleteKey(id: string) {
    if (!confirm("Are you sure you want to delete this API key? This action cannot be undone.")) return
    
    loading = true
    await new Promise(resolve => setTimeout(resolve, 500))
    apiKeys = apiKeys.filter(key => key.id !== id)
    loading = false
  }
</script>

<svelte:head>
  <title>API Keys</title>
</svelte:head>

<div class="flex justify-between items-center mb-6">
  <h1 class="text-2xl font-bold">API Keys</h1>
  <button 
    class="btn btn-primary" 
    onclick={openCreateModal}
    disabled={loading}
  >
    Create New Key
  </button>
</div>

{#if apiKeys.length === 0}
  <div class="card bg-base-100 shadow-sm border p-8 text-center">
    <div class="mb-4">
      <svg class="w-16 h-16 mx-auto text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1121 9z" />
      </svg>
    </div>
    <h3 class="text-lg font-semibold mb-2">No API keys yet</h3>
    <p class="text-gray-600 mb-4">Create your first one to get started.</p>
    <button class="btn btn-primary" onclick={openCreateModal}>
      Create Your First API Key
    </button>
  </div>
{:else}
  <div class="space-y-4">
    {#each apiKeys as key (key.id)}
      <div class="card bg-base-100 shadow-sm border p-6">
        <div class="flex justify-between items-start">
          <div class="flex-1">
            <div class="flex items-center gap-3 mb-2">
              <h3 class="text-lg font-semibold">{key.name}</h3>
              <div class="badge {key.status === 'Active' ? 'badge-success' : 'badge-error'} badge-sm">
                {key.status}
              </div>
            </div>
            <div class="text-sm text-gray-600 mb-1">
              <span class="font-mono bg-gray-100 px-2 py-1 rounded text-xs">{key.keyPrefix}</span>
            </div>
            <div class="text-sm text-gray-500">
              Created: {new Date(key.created).toLocaleDateString()}
            </div>
          </div>
          <div class="flex gap-2">
            <button 
              class="btn btn-outline btn-sm"
              onclick={() => copyToClipboard(key.keyPrefix)}
              title="Copy key prefix"
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
              </svg>
              Copy
            </button>
            <button 
              class="btn btn-error btn-outline btn-sm"
              onclick={() => deleteKey(key.id)}
              disabled={loading}
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
              </svg>
              Delete
            </button>
          </div>
        </div>
      </div>
    {/each}
  </div>
{/if}

{#if showCreateModal}
  <div class="modal modal-open">
    <div class="modal-box">
      {#if !showSuccess}
        <h3 class="font-bold text-lg mb-4">Create New API Key</h3>
        
        <div class="form-control mb-4">
          <label class="label" for="keyName">
            <span class="label-text">Key Name</span>
          </label>
          <input 
            id="keyName"
            type="text" 
            placeholder="e.g., Production API, Development" 
            class="input input-bordered w-full" 
            bind:value={newKeyName}
            disabled={loading}
          />
        </div>

        <div class="modal-action">
          <button class="btn" onclick={closeModal} disabled={loading}>Cancel</button>
          <button 
            class="btn btn-primary" 
            onclick={createApiKey}
            disabled={loading || !newKeyName.trim()}
          >
            {#if loading}
              <span class="loading loading-spinner loading-sm"></span>
              Generating...
            {:else}
              Generate Key
            {/if}
          </button>
        </div>
      {:else}
        <h3 class="font-bold text-lg mb-4">API Key Created Successfully</h3>
        
        <div class="alert alert-warning mb-4">
          <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
          </svg>
          <span>Save this key now - it won't be shown again!</span>
        </div>

        <div class="form-control mb-4">
          <label class="label">
            <span class="label-text font-semibold">Your new API key:</span>
          </label>
          <div class="input-group">
            <input 
              type="text" 
              class="input input-bordered flex-1 font-mono text-sm" 
              value={createdKey}
              readonly
            />
            <button 
              class="btn btn-outline"
              onclick={() => copyToClipboard(createdKey || "")}
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
              </svg>
              Copy
            </button>
          </div>
        </div>

        <div class="modal-action">
          <button class="btn btn-primary" onclick={closeModal}>Done</button>
        </div>
      {/if}
    </div>
    <div class="modal-backdrop" onclick={closeModal}></div>
  </div>
{/if}