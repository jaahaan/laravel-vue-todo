<template>
    <div class="container mt-3 ">
        <div class="row justify-content-center">
            <div class="col-md-8 col-lg-6">
                <div class="card p-5">
                    <div class="card-header text-center text-info">
                        <h4>Todo List</h4>
                    </div>
                    <div class="card-body">
                        <div class="input-group">
                            <input type="text" placeholder="Write Your Task.." class="form-control"
                                v-model="todo_input">
                            <div class="input-group-append">
                                <button type="button" :class="[todo_input? 'yes' : 'no', 'btn btn-dark']" @click="saveTodo()"><i class="fa-solid fa-plus"></i></button>
                            </div>
                        </div>
                        <div>
                            <button type="button" class="btn btn-sm text-danger float-right d-block"
                                @click="resetTodo()">Reset</button>
                        </div>
                    </div>
                    <div class="container justify-content-center">
                        <nav>
                            <div class="m-auto nav nav-tabs" id="nav-tab" role="tablist">
                                <button class="nav-link m-auto show active" id="nav-up-tab" data-bs-toggle="tab"
                                    data-bs-target="#nav-up" type="button" role="tab" aria-controls="nav-up"
                                    aria-selected="false">Upcoming Tasks</button>
                                <button class="nav-link m-auto" id="nav-cm-tab" data-bs-toggle="tab"
                                    data-bs-target="#nav-cm" type="button" role="tab" aria-controls="nav-cm"
                                    aria-selected="false">Completed Tasks</button>
                            </div>
                        </nav>
                    </div>
                    <div class="tab-content" id="nav-tabContent">
                        <div class="tab-pane fade show active" id="nav-up" role="tabpanel" aria-labelledby="nav-up-tab">
                            <h4 class="alert-danger py-3 text-center mt-5">
                                Upcoming Tasks
                            </h4>
                            <table class="table table-striped rounded">

                                <thead>
                                    <th class="text-center">Title</th>
                                    <th class="text-center">Action</th>
                                </thead>
                                <tbody>
                                    <tr v-for="(todo, index) in todos" :key="index" v-if="todo.completed == 'No'">
                                        <template v-if="edit_todo_id == todo.id">
                                            <td>
                                                <input type="text" class="form-control" v-model="edit_input" :ref="`todo${todo.id}`"/>
                                            </td>
                                            <td class="text-center">
                                                <button :class="[edit_input==todo.title? 'notEdited' : 'edited', 'btn ']" @click="updateTodo()">
                                                    <i class="fa-solid fa-square-check"></i>
                                                </button>
                                                <button class="btn btn-sm text-danger" @click="resetTodo()">
                                                    <i class="fa-solid fa-xmark"></i>
                                                </button>
                                            </td>
                                        </template>
                                        <template v-else>
                                            <td>
                                                <button type="button" class="btn btn-sm text-dark"
                                                    @click="updateCheck(index)">
                                                    <i class="fa-solid fa-circle-check"></i> </button> {{
                                                            todo.title
                                                    }}
                                            </td>

                                            <td class="text-center">

                                                <button type="button" class="btn btn-sm text-info"
                                                    @click="editTodo(index)"><i
                                                        class="fa-solid fa-pen-to-square"></i></button>
                                                <button type="button" class="btn btn-sm text-danger"
                                                    @click="deleteTodo(index)"><i
                                                        class="fa-solid fa-trash-can "></i></button>
                                            </td>
                                        </template>
                                    </tr>
                                </tbody>

                            </table>
                        </div>

                        <div class="tab-pane fade" id="nav-cm" role="tabpanel" aria-labelledby="nav-cm-tab">
                            <h4 class="alert-success py-3 text-center mt-5">
                                Completed Tasks
                            </h4>
                            <table class="table table-dark table-striped">
                                <thead>
                                    <th class="text-center">Title</th>
                                    <th class="text-center">Action</th>
                                </thead>
                                <tbody>
                                    <tr v-for="(todo, index) in todos" :key="index" v-if="todo.completed == 'Yes'">
                                        <td><button type="button" class="btn btn-sm text-success"
                                                @click="removeCheck(index)">
                                                <i class="fa-solid fa-circle-check"></i> </button> {{
                                                        todo.title
                                                }}</td>
                                        <td>
                                            <button type="button" class="btn btn-sm text-danger d-block m-auto"
                                                @click="deleteTodo(index)"><i
                                                    class="fa-solid fa-trash-can "></i></button>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

</template>

<script>
export default {
    data() {
        return {
            todos: [],
            api: 'http://localhost:8000/api/todos',
            todo_input: '',
            edit_todo_id: '',
            edit_index: '',
            edit_input: ''
        }
    },
    mounted() {
        this.axios.get(this.api).then(res => {
            this.todos = res.data;
        });
        console.log('Component mounted.');
    },
    methods: {
        saveTodo() {
            if (this.todo_input.length > 0) {
                let data = { 'title': this.todo_input };
                this.axios.post(this.api, data).then(res => {
                    this.todo_input = '';
                });
                this.axios.get(this.api).then(res => {
                    this.todos = res.data;
                });
            }
        },

        deleteTodo(index) {
            if (this.todos[index].id) {
                this.axios.delete(this.api + '/' + this.todos[index].id).then(res => {
                    this.todos.splice(index, 1);
                });
            }
        },
        editTodo(index) {
            if (this.todos[index].id) {
                this.edit_input = this.todos[index].title;
                this.edit_todo_id = this.todos[index].id;
                this.edit_index = index;
                this.$nextTick(() => {
                    if (this.$refs["todo" + this.todos[index].id]) {
                        this.$refs["todo" + this.todos[index].id][0].focus();
                    }
                });
            }
        },
        updateTodo() {
            if (this.edit_input.length > 0) {
                let data = { 'title': this.edit_input };
                this.axios.patch(this.api + '/' + this.todos[this.edit_index].id, data).then(res => {
                    this.todos[this.edit_index].title = res.data.title;
                    this.resetTodo();
                });
            }
        },
        updateCheck(index) {
            let data = { 'completed': 'Yes' };
            this.axios.patch(this.api + '/' + this.todos[index].id, data).then(res => {
                this.resetTodo();
                this.todos.splice(index, 1);
                this.todos.push(res.data);
            });
        },
        removeCheck(index) {
            let data = { 'completed': 'No' };
            this.axios.patch(this.api + '/' + this.todos[index].id, data).then(res => {
                this.resetTodo();
                this.todos.splice(index, 1);
                this.todos.push(res.data);
            });
        },
        resetTodo() {
            this.todo_input = '';
            this.edit_input = '';
            this.edit_todo_id = '';
            this.edit_index = '';
            this.index = '';
        }
    }
}
</script>

<style scoped>
.yes{
    background-color: #00C25E;
}
.no{
    background-color: #292b2c;
}
.edited{
    color: #00C25E;
}
.notEdited{
    color: #292b2c;
}
</style>