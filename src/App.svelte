<script>
  import { onMount } from 'svelte';

  let users = [];
  let name = '';
  let email = '';
  let id = '';

  // Fetch users
  async function fetchUsers() {
    try {
      const response = await fetch('http://localhost:3000/graphql', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          query: `
            query {
              users {
                id
                name
                email
              }
            }
          `,
        }),
      });
      const data = await response.json();
      users = data.data.users;
    } catch (error) {
      console.error('Error fetching users:', error);
    }
  }

  // Create a new user
  async function createUser() {
    try {
      const response = await fetch('http://localhost:3000/graphql', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          query: `
            mutation {
              createUser(name: "${name}", email: "${email}") {
                id
                name
                email
              }
            }
          `,
        }),
      });
      const data = await response.json();
      users = [...users, data.data.createUser];
      name = '';
      email = '';
    } catch (error) {
      console.error('Error creating user:', error);
    }
  }

  // Update a user
  async function updateUser() {
    try {
      const response = await fetch('http://localhost:3000/graphql', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          query: `
            mutation {
              updateUser(id: "${id}", name: "${name}", email: "${email}") {
                id
                name
                email
              }
            }
          `,
        }),
      });
      const data = await response.json();
      users = users.map(user => user.id === id ? data.data.updateUser : user);
      name = '';
      email = '';
      id = '';
    } catch (error) {
      console.error('Error updating user:', error);
    }
  }

  // Delete a user
  async function deleteUser() {
    try {
      await fetch('http://localhost:3000/graphql', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          query: `
            mutation {
              deleteUser(id: "${id}")
            }
          `,
        }),
      });
      users = users.filter(user => user.id !== id);
      id = '';
    } catch (error) {
      console.error('Error deleting user:', error);
    }
  }

  // Fetch users on component mount
  onMount(async () => {
    await fetchUsers();
  });
</script>

<h1>Users</h1>

<ul>
  {#each users as user}
    <li>
      <span>{user.name}</span>
      <span>{user.email}</span>
      <button on:click={() => {
        id = user.id;
        name = user.name;
        email = user.email;
      }}>Edit</button>
      <button on:click={() => {
        id = user.id;
        deleteUser();
      }}>Delete</button>
    </li>
  {/each}
</ul>

<!-- Create User Form -->
<form on:submit|preventDefault={createUser}>
  <input type="text" bind:value={name} placeholder="Name" required />
  <input type="email" bind:value={email} placeholder="Email" required />
  <button type="submit">Create</button>
</form>

<!-- Update User Form -->
{#if id}
  <h2>Edit User</h2>
  <form on:submit|preventDefault={updateUser}>
    <input type="hidden" bind:value={id} />
    <input type="text" bind:value={name} placeholder="Name" required />
    <input type="email" bind:value={email} placeholder="Email" required />
    <button type="submit">Update</button>
  </form>
{/if}
