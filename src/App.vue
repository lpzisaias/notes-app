<script setup>
import { onMounted, reactive, ref, watch, computed } from "vue";
import { Plus, Delete, Edit } from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
const notes = ref([]);
const dialogVisible = ref(false);
const formRef = ref();
const form = ref({
  title: "",
  content: "",
});
const noteColor = ref("#9EFFFF");
const dataIndex = ref(null);

const rules = reactive({
  title: [
    {
      required: true,
      message: "Please input title",
      trigger: "blur",
    },
  ],
  content: [
    {
      required: true,
      message: "Please input content",
      trigger: "blur",
    },
  ],
});

const noteColors = ref([
  {
    name: "Blue",
    code: "#9EFFFF",
  },
  {
    name: "Purple",
    code: "#B69CFF",
  },
  {
    name: "Yellow",
    code: "#FFF599",
  },
  {
    name: "Green",
    code: "#91F48F",
  },
  {
    name: "Pink",
    code: "#FD99FF",
  },
  {
    name: "Red",
    code: "#FF9E9E",
  },
]);

// Sorts items in ascending order by creation date
const notes_asc = computed(() =>
  notes.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

//MARK: New note
const newNote = () => {
  console.log(noteColors.value);
  if (dataIndex.value != null) {
    resetForm();
  }
  dataIndex.value = null;
  dialogVisible.value = true;
  // formRef.value.clearValidate();
};

//MARK: Save note
const saveNote = async (formEl) => {
  if (!formEl) return;
  await formEl.validate((valid) => {
    if (valid) {
      if (dataIndex.value === null) {
        notes.value.push({
          title: form.value.title,
          content: form.value.content,
          color: noteColor.value,
          createdAt: new Date().getTime(),
        });
        ElMessage({
          message: "Note added successfully.",
          type: "success",
        });
      } else {
        notes.value[dataIndex.value].title = form.value.title;
        notes.value[dataIndex.value].content = form.value.content;
        notes.value[dataIndex.value].color = noteColor.value;
        ElMessage({
          message: "Note edited successfully.",
          type: "success",
        });
      }

      dialogVisible.value = false;
      resetForm();
    } else {
      return;
    }
  });
};

//MARK: Edit note
const editNote = (data, index) => {
  dataIndex.value = index;
  dialogVisible.value = true;
  form.value.title = data.title;
  form.value.content = data.content;
  noteColor.value = data.color;
  formRef.value.clearValidate();
};

watch(
  notes,
  (newValue) => {
    localStorage.setItem("notes", JSON.stringify(newValue));
  },
  { deep: true }
);

//MARK: Delete note
const deleteNote = (index) => {
  notes.value.splice(index, 1);
  ElMessage({
    message: "Note deleted successfully.",
    type: "success",
  });
};
const resetForm = () => {
  form.value.title = "";
  form.value.content = "";
};

onMounted(() => {
  notes.value = JSON.parse(localStorage.getItem("notes")) || [];
});
</script>

<template>
  <div>
    <!-- // MARK: Header -->
    <el-row align="middle" class="header-row">
      <el-col :span="12">
        <h1>My Notes</h1>
      </el-col>
      <el-col class="button-col" :span="12" align="right">
        <el-button
          type="primary"
          :icon="Plus"
          circle
          size="large"
          @click="newNote"
        ></el-button>
      </el-col>
    </el-row>
    <br />
    <!-- //MARK: Card container -->
    <el-row :gutter="20" class="note-container">
      <el-col
        v-for="(note, index) in notes_asc"
        :key="note.title"
        :xs="24"
        :sm="12"
        :md="8"
        :lg="6"
      >
        <el-card
          shadow="always"
          :body-style="{ padding: '20px' }"
          class="box-card"
          :style="{ 'background-color': note.color }"
        >
          <template #header>
            <el-row align="middle">
              <el-col :span="12"
                ><h3>{{ note.title }}</h3></el-col
              >
              <el-col :span="12" align="right"
                ><el-button
                  type="warning"
                  :icon="Edit"
                  @click="editNote(note, index)"
                ></el-button>
                <el-button
                  type="danger"
                  :icon="Delete"
                  @click="deleteNote(index)"
                ></el-button
              ></el-col>
            </el-row>
          </template>
          {{ note.content }}
        </el-card>
      </el-col>
    </el-row>
    <!-- //MARK: Dialog Form -->
    <el-dialog
      v-model="dialogVisible"
      width="80%"
      :close-on-click-modal="false"
    >
      <template #header>
        <h2>New Note</h2>
      </template>
      <span>
        <!-- //MARK: Form -->
        <el-form
          :model="form"
          ref="formRef"
          label-width="80px"
          label-position="top"
          :rules="rules"
        >
          <el-form-item label="Title" prop="title">
            <el-input
              v-model="form.title"
              placeholder="Type your title"
            ></el-input>
          </el-form-item>
          <el-form-item label="Content" prop="content">
            <el-input
              type="textarea"
              v-model="form.content"
              placeholder="Type your note content"
              rows="4"
            ></el-input>
          </el-form-item>
          <el-form-item label="Color">
            <el-radio-group v-model="noteColor">
              <el-radio-button
                v-for="color in noteColors"
                :key="color.name"
                :label="color.code"
                :style="{
                  '--el-radio-button-checked-bg-color': color.code,
                  '--el-radio-button-checked-text-color': '#252525',
                  '--el-radio-button-checked-border-color': color.code,
                }"
              >
                {{ color.name }}
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </span>
      <template #footer>
        <span>
          <el-button @click="dialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="saveNote(formRef)">OK</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #fff;
  position: relative;
  height: 100%;
  top: 0px;
  bottom: 0px;
}

.header-row {
  padding: 0px 1rem;
}
.button-col {
  margin: auto;
}

.note-container .el-col {
  margin-bottom: 20px;
}

.el-card {
  border: none;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.el-card__header {
  border-bottom: 1px solid #3b3b3b;
}

.box-card {
  max-height: 350px;
  overflow: auto;
}
.el-dialog {
  background-color: #252525;
}

.el-form-item__label {
  color: #fff;
}
.el-input {
  --el-input-border-color: auto;
}
.el-input__wrapper {
  background-color: #3b3b3b;
}
.el-input__inner {
  background-color: #3b3b3b;
  color: #fff;
}

.el-textarea {
  --el-input-border-color: auto;
}

.el-textarea__inner {
  background-color: #3b3b3b;
  color: #fff;
}

.el-radio-button__inner {
  background: #3b3b3b;
  color: #fff;
  border-color: #545454;
}
.el-radio-button:first-child .el-radio-button__inner {
  border-left-color: #3b3b3b;
}
</style>
