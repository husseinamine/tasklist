<script>
  import { Modal, Dropdown } from "bootstrap";
  import { onMount } from "svelte";

  $: tasks = []

  $: filteredTasks = []
  $: searchQuery = ""
  $: {
    filteredTasks = filterTasks(tasks, searchQuery)
  }

  let taskModal
  let taskId
  let taskTitle = ""
  let taskDesc = ""

  onMount(() => {
    taskModal = new Modal(`#taskModal`)

    let taskIdString = localStorage.getItem("taskId") || "0"
    taskId = parseInt(taskIdString)

    let tasksString = localStorage.getItem("tasks") || "[]"
    tasks = JSON.parse(tasksString)
  })

  const markTask = (dId, id, status) => {
    let d = new Dropdown(dId)
    d.hide()

    let taskIndex = tasks.findIndex((task) => task.id == id)
    tasks[taskIndex].status = status
    tasks = tasks
  }

  const statusIcon = (status) => {
    switch (status) {
      case 0:
        return "🖋️"
      case 1:
        return "⚒️"
      case 2:
        return "✅"
    }
  }

  const statusClass = (status) => {
    switch (status) {
      case 0:
        return "btn btn-secondary dropdown-toggle"
      case 1:
        return "btn btn-primary dropdown-toggle"
      case 2:
        return "btn btn-success dropdown-toggle"
    }
  }

  const addTask = () => {
    if (taskTitle == "" || taskDesc == "") {
      return
    }
    taskModal.hide()

    taskId++

    tasks = [
      {
        title: taskTitle,
        desc: taskDesc,
        id: taskId,
        status: 0,
      },
      ...tasks
    ]

    taskTitle = ""
    taskDesc = ""

    localStorage.setItem("taskId", taskId.toString())
    localStorage.setItem("tasks", JSON.stringify(tasks))
  }

  const deleteTask = (id) => {
    tasks = tasks.filter(task => task.id != id)
    localStorage.setItem("tasks", JSON.stringify(tasks))
  }

  const filterTasks = (tasks, query) => {
    query = query.toLowerCase()
    
    return tasks.filter(task => {
      let taskTitle = task.title.toLowerCase()
      let taskDesc = task.desc.toLowerCase()

      return (taskTitle.includes(query) || taskDesc.includes(query))
    })
  }

</script>

<main>
  <br />
  <div class="row">
    <div class="col-8">
      <h1>📝 Tasklist</h1>
    </div>
    <div class="d-grid col-4">
      <button 
        on:click={() => taskModal.show()} 
        class="btn btn-lg btn-success"
      >
        Add a task
      </button>
    </div>
  </div>
  <br />
  <div class="input-group flex-nowrap">
    <span class="input-group-text" id="addon-wrapping">🔍</span>
    <input bind:value={searchQuery} type="text" class="form-control" placeholder="I'm looking for..." aria-label="Search" aria-describedby="addon-wrapping">
  </div>
  <hr />
  <div>
    {#each filteredTasks as task}
      {#key task.id}
      <div class="card" style="margin-bottom: 8px;">
        <div class="row">
          <div class="col">
            <div class="card-body">
              <h5 class="card-title">{task.title}</h5>
              <p class="card-text">{task.desc}</p>
              <div class="dropdown" id={`dd${task.id}`}>
                <button class={statusClass(task.status)} type="button" data-bs-toggle="dropdown" aria-expanded="false">
                  {statusIcon(task.status)} Mark as
                </button>
                <ul class="dropdown-menu">
                  <li><button class="dropdown-item" on:click={(e) => markTask(`#dd${task.id}`, task.id, 0)}>🖋️ To-Do</button></li>
                  <li><button class="dropdown-item" on:click={(e) => markTask(`#dd${task.id}`, task.id, 1)}>⚒️ Doing</button></li>
                  <li><button class="dropdown-item" on:click={(e) => markTask(`#dd${task.id}`, task.id, 2)}>✅ Done</button></li>
                </ul>
              </div>
            </div>
          </div>
          <div class="col-auto row align-items-center">
            <div class="col">
              <div class="card-body">
                <button class="btn btn-outline-danger" on:click={() => deleteTask(task.id)}>
                  <strong>Remove</strong>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      {/key}
    {/each}
    </div>
    <div class="modal fade" id="taskModal" tabindex="-1" aria-labelledby="taskModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="taskModalLabel">Add a task</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <form>
              <div class="mb-3">
                <label for="task-title" class="col-form-label">Title</label>
                <input required bind:value={taskTitle} placeholder="Example: Buy a car" type="text" class="form-control" id="task-title">
              </div>
              <div class="mb-3">
                <label for="task-desc" class="col-form-label">Description</label>
                <textarea required bind:value={taskDesc} placeholder="Example: Volvo, KIA, etc." class="form-control" id="task-desc"></textarea>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
            <button type="button" class="btn btn-success" on:click={addTask}>Add Task</button>
          </div>
        </div>
      </div>
    </div>
</main>

<style>

</style>
