<script>
	import TodoList from './lib/TodoList.svelte';
	import { v4 as uuid } from 'uuid';
	import { tick, onMount } from 'svelte';

	let todoList;
	let showList = true;

	let todos = null;
	let error = null;
	let isLoading = false;
	let isAdding = false;
	let disabledItems = [];

	onMount(() => {
		loadTodos();
	});

	async function loadTodos() {
		isLoading = true;
		await fetch('https://jsonplaceholder.typicode.com/todos?_limit=10').then(
			async (response) => {
				if (response.ok) {
					todos = await response.json();
				} else {
					error = 'An error has occurred';
				}
			}
		);
		isLoading = false;
	}

	async function handleAddTodo(event) {
		event.preventDefault();
		isAdding = true;
		await fetch('https://jsonplaceholder.typicode.com/todos', {
			method: 'POST',
			body: JSON.stringify({
				title: event.detail.title,
				completed: false
			}),
			headers: {
				'Content-type': 'application/json; charset=UTF-8'
			}
		}).then(async (response) => {
			if (response.ok) {
				const todo = await response.json();
				todos = [...todos, { ...todo, id: uuid() }];
				todoList.clearInput();
			} else {
				alert('An error has occurred.');
			}
		});
		isAdding = false;
		await tick();
		todoList.focusInput();
	}

	async function handleRemoveTodo(event) {
		const id = event.detail.id;
		if (disabledItems.includes(id)) return;
		disabledItems = [...disabledItems, id];
		await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'DELETE'
		}).then((response) => {
			if (response.ok) {
				todos = todos.filter((t) => t.id !== event.detail.id);
			} else {
				alert('An error has occurred.');
			}
		});
		disabledItems = disabledItems.filter((itemId) => itemId !== id);
	}

	function handleToggleTodo(event) {
		todos = todos.map((todo) => {
			if (todo.id === event.detail.id) {
				return { ...todo, completed: event.detail.value };
			}
			return { ...todo };
		});
	}
</script>

<label>
	<input type="checkbox" bind:checked={showList} />
	Show/Hide list
</label>
{#if showList}
	<div style:max-width="400px">
		<TodoList
			{todos}
			{error}
			{isLoading}
			{disabledItems}
			disableAdding={isAdding}
			bind:this={todoList}
			on:addtodo={handleAddTodo}
			on:removetodo={handleRemoveTodo}
			on:toggletodo={handleToggleTodo}
			let:todo
			let:handleToggleTodo
      let:index
		>

    <!-- <svelte:fragment slot="title">{index + 1}- {todo.title}</svelte:fragment> -->

    <div slot="index" let:index>
      
      {index + 1} - {todo.title}
    
  </div>
      
			<!-- <Todo {todo} on:remove on:toggle/> -->

			<!-- <div>
				<input
					disabled={disabledItems.includes(id)}
					on:input={(event) => {
						event.currentTarget.checked = completed;
						handleToggleTodo(id, !completed);
					}}
					type="checkbox"
					checked={completed}
				/>
          {title}
				{todo.title}
			</div> -->
		</TodoList>
	</div>
{/if}

<style>
</style>
