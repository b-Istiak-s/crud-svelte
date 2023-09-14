<script>
    import { initializeApp } from "firebase/app";
    import { getFirestore, collection, setDoc, getDocs, doc, updateDoc, onSnapshot, QuerySnapshot, orderBy, query, deleteDoc } from "firebase/firestore";
    import { firebaseConfig } from "../lib/firebaseConfig";

    const firebaseApp = initializeApp(firebaseConfig);
    const db = getFirestore();

    /**
	 * @type {any[]}
	 */
    let todos = [];
    let isLoading = true;
    const colRef = collection(db, "todos");

    const unsubscribe = onSnapshot(query(colRef, orderBy("createdAt", "asc")), (QuerySnapshot) => {
        /**
		 * @type {any[]}
		 */
        let fbTodos = [];
        QuerySnapshot.forEach((doc) => {
            let todo = { id: doc.id, ...doc.data() };
            fbTodos = [todo, ...fbTodos];
        });
        todos = fbTodos;
        isLoading = false;
    })

    let task = "";

    const addTodo = async () => {
        const todosCollection = doc(collection(db, "todos"));
        await setDoc(todosCollection, {
            task: task,
            isComplete: false,
            createdAt: new Date()
        });
        task = "";
    }
    const removeItem = (/** @type {any} */ _item) => {
        deleteDoc(doc(db, "todos", _item.id));
    }
    const markTodoAsComplete = (/** @type {any} */ _item) => {
        updateDoc(doc(db, "todos", _item.id), {
            isComplete: !_item.isComplete
        });
    }
</script>

{#if isLoading}
    <p>Loading...</p> <!-- Display a loading message while data is being fetched -->
{:else}
    <input type="text" name="todo" id="todo" placeholder="Add a todo" bind:value={task}> 
    <button on:click={addTodo}>Add</button>
    <ol>
        {#each todos as todo}
            <li class:complete={todo.isComplete}>
                <span>
                    Task : {todo.task}; Created at : {todo.createdAt.toDate().toString()}
                </span>
                <span>
                    <button on:click={() => markTodoAsComplete(todo)}>✔</button>
                    <button on:click={() => removeItem(todo)}>✗</button>
                </span>
            </li>
        {:else}
            <p>No item found</p>
        {/each}
    </ol>
{/if}

<style>
    .complete {
        text-decoration: line-through;
    }
</style>