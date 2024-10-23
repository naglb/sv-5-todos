<script lang="ts">
    type Todo = {
        text: string;
        done: boolean;
    }

    type Filters = 'all' | 'active' | 'completed';
    
    let todos = $state<Todo[]>([])
    let filter = $state<Filters>('all')
    let filteredTodos = $derived(filterTodos())

    $effect(() => {
		const savedTodos = localStorage.getItem('todos')
		savedTodos && (todos = JSON.parse(savedTodos))
	})

    $effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos))
	})

    function addTodo(event: KeyboardEvent){
        if (event.key !== 'Enter') return;

        const todoEl = event.target as HTMLInputElement;
        const text: string = todoEl.value.trim();
        const done = false;

        // Only adding a todo if the space isn't empty
        if (text) {
            todos = [...todos, { text, done }]
        }

        todoEl.value = '';
    }

    function editTodoText(index: number) {
        const newText = prompt("Edit todo", todos[index].text);
        if (newText !== null) {
            todos[index].text = newText.trim();
    }
}

    function deleteTodo(index: number) {
        todos = todos.filter((_, i) => i !== index);
    }

    function toggleTodo(event: Event) {
        const inputEl = event.target as HTMLInputElement;
        const index = +inputEl.dataset.index!;
        todos[index].done = !todos[index].done;
    }

    function setFilter (newFilter: Filters) {
        filter = newFilter
    }

    function filterTodos() {
        switch(filter) {
            case 'all':
                return todos;
            case 'active':
                return todos.filter(todo => !todo.done)
            case 'completed':
                return todos.filter(todo => todo.done)
        }
    }

    function remaining() {
        return todos.filter(todo => !todo.done).length;
    }

    // Editing the header
    let headerText = $state<string>("The todo list.");
    let editingHeader = $state<boolean>(false);

    function editHeader() {
        editingHeader = true; // Set editing mode to true
    }

    function saveHeader(event: KeyboardEvent) {
        if (event.key === 'Enter') {
            editingHeader = false; // Exit editing mode
        }
    }

    function blurHeader() {
        editingHeader = false; // Exit editing mode on blur
    }

</script>

<button class="header" onclick={editHeader} aria-label="Edit header" aria-haspopup="true">
    {#if editingHeader}
        <input 
            type="text" 
            bind:value={headerText} 
            onkeydown={saveHeader} 
            onblur={blurHeader}
            class="header-input"
        />
    {:else}
        <span class="header-text">{headerText}</span>
    {/if}
</button>

<div class="todo-input">
    <input onkeydown={addTodo} placeholder="Add todo" type="text" class="todo-text" />
</div>

<div class="todos">
    {#each filteredTodos as todo, i}
        <div class="todo">
            <input onchange={toggleTodo} data-index={i} checked={todo.done} type="checkbox" class="done-btn">
            <span class="task-text" class:completed={todo.done}>{todo.text}</span>
            <button class="delete-btn" onclick={() => deleteTodo(i)}>
                <img src="/assets/images/trash-bin-trash-svgrepo-com.svg" alt="Delete">
            </button>
            <button class="edit-btn" onclick={() => editTodoText(i)}>
                <img src="/assets/images/edit-svgrepo-com.svg" alt="Edit">
            </button>
        </div>
    {/each}
</div>

<div class="filters">
    <button onclick={() => setFilter('all')} class="btn-all">All</button>
	<button onclick={() => setFilter('active')} class="btn-active">Active</button>
	<button onclick={() => setFilter('completed')} class="btn-completed">Completed</button>
</div>

<p class="p">{remaining()} tasks remaining</p>

<style>
    @import 'src/styles/todolist.css';
</style>