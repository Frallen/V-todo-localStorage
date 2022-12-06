<template>
  <div class="container">
    <div class="task">
      <h3 class="task-title">Todo app</h3>
      <div class="task-wrapper">
        <!--Пришлось через v-show тк идет рередер и нужна повторная инициализация-->
        <ul v-show="todo.length" class="collapsible task-list z-depth-2">
          <li
            class="task-list-item"
            v-for="item in todo"
            :key="item.id"
            :data-id="item.id"
          >
            <div
              class="collapsible-header"
              :class="{ done: item.status === 'Готово' }"
            >
              <label>
                <input type="checkbox" @change="TodoStatus" />
                <span></span>
              </label>
              {{ item.todoName }}
              <div class="chip">
                {{ item.status }}
              </div>
            </div>
            <div class="collapsible-body">
              <div class="collapsible-body-head">
                <div>Выполнить до {{ item.todoDate }}</div>
                <div class="actions">
                  <div class="delete red-text" @click.stop="DeleteToDo">
                    <i class="material-icons">close</i>Удалить
                  </div>
                  <div class="edit green-text" @click.stop="EditToDo(item)">
                    <i class="material-icons">edit</i>Редактировать
                  </div>
                </div>
              </div>
              <p>{{ item.todoText }}</p>
            </div>
          </li>
        </ul>
        <div v-show="!todo.length" class="task-wrapper-empty grey z-depth-2">
          <h5>Список задач пуст</h5>
        </div>
        <Form
          class="task-form z-depth-2"
          @submit="SetToTo"
          :validation-schema="schema"
          ref="form"
        >
          <div class="input-field">
            <Field name="todoName" id="todoName" v-slot="{ field }" type="text">
              <input
                type="text"
                id="title"
                class="validate"
                v-bind="field"
              /><label for="title">Название задачи</label>
            </Field>

            <ErrorMessage name="todoName" class="red-text" />
          </div>
          <div class="input-field">
            <Field name="todoDate" id="todoDate" v-slot="{ field }" type="text">
              <input
                type="text"
                class="datepicker validate"
                ref="Date"
                v-bind="field"
                id="date"
              />
              <label for="date">Выберите дату выполения</label>
            </Field>
            <ErrorMessage name="todoDate" class="red-text" />
          </div>
          <div class="input-field">
            <Field
              name="todoText"
              id="todoText"
              v-slot="{ field }"
              type="text-area"
            >
              <textarea
                class="materialize-textarea"
                id="text"
                v-bind="field"
              ></textarea
              ><label for="text">Описание задачи</label></Field
            >
            <ErrorMessage name="todoText" class="red-text" />
          </div>
          <button class="btn blue darken-2">
            {{ formStatus === 0 ? "Создать задачу" : "Обновить задачу" }}
          </button>
        </Form>
      </div>
    </div>
  </div>
</template>

<script>
import M from "materialize-css/dist/js/materialize.min";
import { Form, Field, ErrorMessage } from "vee-validate";
import * as yup from "yup";
export default {
  name: "App",
  components: { Form, Field, ErrorMessage },
  data() {
    return {
      formStatus: 0,
      optionsDate: {
        format: "dd.mm.yyyy",
      },
      todo: [],
      updatedId: "",
    };
  },
  mounted() {
    M.AutoInit();
    M.Datepicker.init(this.$refs.Date, this.optionsDate);
    this.todo = JSON.parse(localStorage.getItem("todo")) ?? [];
  },
  computed: {
    schema() {
      return yup.object({
        todoName: yup.string().required(),
        todoText: yup.string().required(),
        todoDate: yup.string().required(),
      });
    },
  },
  watch: {
    todo() {
      localStorage.setItem("todo", JSON.stringify(this.todo));
    },
  },
  methods: {
    SetToTo(data, { resetForm }) {
      if (this.formStatus === 0) {
        data.status = "В работе";
        data.id = Math.random().toString();
        this.todo.push(data);
        localStorage.setItem("todo", JSON.stringify(this.todo));
        resetForm();
      } else {
        this.todo.map((p) => {
          if (p.id === this.updatedId) {
            p.todoDate = data.todoDate;
            p.todoName = data.todoName;
            p.todoText = data.todoText;
            p.status = "В работе";
          }
        });
        this.updatedId = null;
        this.formStatus = 0;
        resetForm();
      }
    },
    TodoStatus(e) {
      let parent = e.currentTarget.parentNode.parentNode.parentNode;
      if (e.currentTarget.checked) {
        this.todo.map((p) => {
          if (p.id.toString() === parent.dataset.id) {
            p.status = "Готово";
          }
        });
      } else {
        this.todo.map((p) => {
          if (p.id.toString() === parent.dataset.id) {
            p.status = "В работе";
          }
        });
      }
    },
    DeleteToDo(e) {
      this.todo = [
        ...this.todo.filter(
          (p) =>
            p.id.toString() !==
            e.currentTarget.parentNode.parentNode.parentNode.parentNode.dataset
              .id
        ),
      ];
      this.updatedId = null;
      this.formStatus = 0;
    },
    EditToDo(item) {
      this.formStatus = 1;
      this.$refs.form.setFieldValue("todoText", item.todoText);
      this.$refs.form.setFieldValue("todoName", item.todoName);
      this.$refs.form.setFieldValue("todoDate", item.todoDate);

      this.updatedId = item.id;
      M.Datepicker.getInstance(this.$refs.Date).setDate(item.todoDate);
    },
  },
};
</script>

<style lang="less">
@import "modern-normalize/modern-normalize.css";
@import "materialize-css/dist/css/materialize.min.css";
@import "_var";
body {
  font-family: "Roboto", sans-serif;
}
.task {
  &-title {
    text-align: center;
  }
  &-wrapper {
    display: flex;
    justify-content: space-between;
    @media @md {
      flex-direction: column;
    }
    &-empty {
      text-align: center;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 45%;
      color: #fff;
      @media @md {
        width: 100%;
      }
    }
  }
  &-list {
    .br(5px);
    width: 45%;
    height: fit-content;
    overflow: auto;
    max-height: 480px;
    @media @md {
      width: 100%;
    }
    p {
      display: block;
      word-break: break-all;
    }
  }
  &-list::-webkit-scrollbar-thumb {
    .br(5px);
    background: #000000a8;
  }
  &-list::-webkit-scrollbar-track {
    background: #00000047;
    .br(5px);
  }
  &-list::-webkit-scrollbar {
    width: 9px;
  }
  &-form {
    display: flex;
    flex-direction: column;
    @media @md {
      margin-top: 1em;
    }
    button {
      border: none;
      outline: none;
      cursor: pointer;
    }
  }

  &-form {
    padding: 1em;
    .br(5px);
    width: 45%;
    height: fit-content;
    @media @md {
      width: 100%;
    }
  }
}
.collapsible-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.collapsible-body-head {
  display: flex;
  justify-content: space-between;
  .actions {
    .delete,
    .edit {
      display: flex;
      align-items: center;
      cursor: pointer;
      i {
        margin-right: 5px;
      }
    }
    .edit {
      margin-top: 10px;
    }
  }
}
.done {
  filter: opacity(0.5);
  text-decoration: line-through;
}
</style>
