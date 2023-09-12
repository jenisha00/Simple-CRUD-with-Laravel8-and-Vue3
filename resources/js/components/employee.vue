<template>
    <div class="container py-5">
        <button
            type="button"
            class="btn btn-primary mb-3"
            data-bs-toggle="modal"
            data-bs-target="#myModal"
        >
            Add Employee
        </button>
        <div
            class="modal fade"
            id="myModal"
            data-bs-backdrop="static"
            data-bs-keyboard="false"
            tabindex="-1"
            aria-labelledby="myModalLabel"
            aria-hidden="true"
        >
            <div class="modal-dialog modal-dialog-centered">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="addModalLabel" v-if="employee_id == ''">
                            Add Employee
                        </h5>
                        <h5 class="modal-title" id="addModalLabel" v-else>
                            Edit Employee
                        </h5>
                        <button
                            type="button"
                            class="btn-close"
                            data-bs-dismiss="modal"
                            aria-label="Close"
                            @click="closeModal"
                        ></button>
                    </div>
                    <div class="modal-body">
                        <ul class="text-sm text-danger" v-if="errors != ''">
                            <li v-for="error in errors" :key="error">
                                <small>{{ error }}</small>
                            </li>
                        </ul>
                        <div class="mb-3">
                            <label for="" class="form-label">Name</label>
                            <input
                                type="text"
                                class="form-control"
                                v-model="form.name"
                            />
                        </div>
                        <div class="mb-3">
                            <label for="" class="form-label">Email</label>
                            <input
                                type="email"
                                class="form-control"
                                v-model="form.email"
                            />
                        </div>
                        <div class="mb-3">
                            <label for="" class="form-label">Phone</label>
                            <input
                                type="number"
                                class="form-control"
                                v-model="form.phone"
                            />
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button
                            type="button"
                            class="btn btn-secondary"
                            data-bs-dismiss="modal"
                            @click="closeModal"
                        >
                            Close
                        </button>
                        <button
                            type="submit"
                            class="btn btn-primary"
                            v-if="employee_id == ''"
                            @click="storeEmployee"
                        >
                            Submit
                        </button>
                        <button
                            type="submit"
                            class="btn btn-primary"
                            v-else
                            @click="updateEmployee"
                        >
                            Update
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <table class="table table-bordered table-striped">
            <thead class="table-dark">
                <tr>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Phone</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <template v-for="employee in employees" :key="employee.id">
                    <tr>
                        <td>{{ employee.name }}</td>
                        <td>{{ employee.email }}</td>
                        <td>{{ employee.phone }}</td>
                        <td>
                            <button
                                type="button"
                                class="btn btn-warning py-1 me-1"
                                style="font-size: 14px"
                                data-bs-toggle="modal"
                                data-bs-target="#myModal"
                                @click="editEmployee(employee)"
                            >
                                Edit
                            </button>
                            <button
                                type="button"
                                class="btn btn-danger py-1"
                                style="font-size: 14px"
                                data-bs-toggle="modal"
                                :data-bs-target="'#deleteModal'+ employee.id"
                            >
                                Delete
                            </button>
                            <div
                                class="modal fade"
                                :id="'deleteModal' + employee.id"
                                data-bs-backdrop="static"
                                data-bs-keyboard="false"
                                tabindex="-1"
                                :aria-labelledby="'deleteModalLabel' + employee.id"
                                aria-hidden="true"
                            >
                                <div class="modal-dialog modal-dialog-centered">
                                    <div class="modal-content">
                                        <div class="modal-header">
                                            <h5 class="modal-title" :id="'deleteModalLabel' + employee.id">
                                                Delete Employee
                                            </h5>
                                            <button
                                                type="button"
                                                class="btn-close"
                                                data-bs-dismiss="modal"
                                                aria-label="Close"
                                            ></button>
                                        </div>
                                        <div class="modal-body">
                                            <h6>Are you sure you want to delete employee '{{ employee.name }}'?</h6>
                                        </div>
                                        <div class="modal-footer">
                                            <button
                                                type="button"
                                                class="btn btn-secondary"
                                                data-bs-dismiss="modal"
                                            >
                                                Close
                                            </button>
                                            <button
                                                type="submit"
                                                class="btn btn-danger"
                                                @click="deleteEmployee(employee.id)"
                                            >
                                                Delete
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </td>
                    </tr>
                </template>
            </tbody>
        </table>
    </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";
import toastr from 'toastr';

export default {
    setup() {
        const employees = ref([]);
        const errors = ref([]);
        const employee_id = ref([]);
        const form = reactive({
            name: "",
            email: "",
            phone: "",
        });

        const getEmployee = async () => {
            let res = await axios.get("api/employees");
            employees.value = res.data;
        };

        const storeEmployee = async () => {
            try {
                let res = await axios.post("api/employees/store", form);
                getEmployee();
                formReset();
                errors.value = '';
                $("#myModal").modal("hide");
                showToastr(Object.keys(res.data)[0],res.data.success);
            } catch (e) {
                if (e.response.status == 422) {
                    var data = [];
                    for (const key in e.response.data.errors) {
                        data.push(e.response.data.errors[key][0]);
                    }
                    errors.value = data;
                }
            }
        };

        const editEmployee = (employee) => {
            employee_id.value = employee.id;
            form.name = employee.name;
            form.email = employee.email;
            form.phone = employee.phone;
        };

        const updateEmployee = async () => {
            try {
                let res = await axios.patch(
                    "api/employees/update/" + employee_id.value,
                    form
                );
                getEmployee();
                formReset();
                errors.value = '';
                $("#myModal").modal("hide");
                showToastr(Object.keys(res.data)[0],res.data.success);
            } catch (e) {
                if (e.response.status == 422) {
                    var data = [];
                    for (const key in e.response.data.errors) {
                        data.push(e.response.data.errors[key][0]);
                    }
                    errors.value = data;
                }
            }
        };

        const closeModal = () => {
            errors.value = '';
            formReset();
        }

        const deleteEmployee = async(id) => {
            let res = await axios.delete('api/employees/delete/' + id);
            getEmployee();
            $(`#deleteModal${id}`).modal('hide');
            showToastr(Object.keys(res.data)[0],res.data.success);
        }

        const showToastr = (type,message) => {
            toastr.options = {
                closeButton: true,
                progressBar: true,
                positionClass: 'toast-top-right',
                timeOut: 3000,
            };
            toastr[type](message);
        }

        onMounted(getEmployee());

        const formReset = () => {
            employee_id.value = "";
            form.name = "";
            form.email = "";
            form.phone = "";
        };

        return {
            employees,
            form,
            storeEmployee,
            errors,
            editEmployee,
            updateEmployee,
            employee_id,
            closeModal,
            deleteEmployee
        };
    },
};
</script>
