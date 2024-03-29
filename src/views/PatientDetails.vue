<template>
  <div class="container">
    <div v-if="patient" class="patient-details">
      <header class="patient-header">
        <h1>{{ patient.name }}, {{ patient.firstname }}</h1>
      </header>

      <section class="patient-info">
        <div class="patient-details-head">
          <span>Age: {{ patient.age }}</span>
          <span>Birth Date: {{ patient.birthDate }}</span>
          <span>Gender: {{ patient.gender }}</span>
        </div>
        <div class="extra-box">
          <span>Note: {{ patient.note || "N/A" }}</span>
        </div>
      </section>

      <section class="patient-todos">
        <h2>Patient-Todos</h2>
        <button class="btn" @click="toggleTodoForm">Add Todo</button>
        <form v-if="showTodoForm" @submit.prevent="addTodo" class="todo-form">
          <label>
            Description:
            <input v-model="newTodo.beschreibung" required />
          </label>
          <label>
            Priority:
            <select v-model="newTodo.prioritaet" required>
              <option value="">Choose One</option>
              <option value="niedrig">Low</option>
              <option value="mittel">Middle</option>
              <option value="hoch">High</option>
            </select>
          </label>
          <button type="submit" class="btn">Add
          </button>
        </form>
        <table class="todo-table">
          <thead>
            <tr>
              <th>Status</th>
              <th>Description</th>
              <th>Priority</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="todo in sortedTodos" :key="todo.toDoId">
              <td>
                <input
                  type="checkbox"
                  :checked="todo.status === 'erledigt'"
                  @change="toggleTodoStatus(todo)"
                />
              </td>
              <td :class="{ 'todo-done': todo.status === 'erledigt' }">
                {{ todo.beschreibung }}
              </td>
              <td>{{ todo.prioritaet }}</td>
              <td>
                <button class="btn btn-delete" @click="deleteTodo(todo.toDoId)">
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <section class="patient-files">
        <h2>Patient-Files</h2>
        <button class="btn" @click="toggleFileForm">Add File </button>
        <form v-if="showForm" @submit.prevent="addFile" class="file-form">
          <label>
            Filename:
            <input v-model="newFile.fileName" required />
          </label>
          <label>
            File:
            <input type="file" v-on:change="handleFileUpload" required />
          </label>
          <label>
            Description:
            <input v-model="newFile.description" required />
          </label>
          <button class="btn">Add File</button>
        </form>
        <table class="file-table">
          <thead>
            <tr>
              <th>Filename</th>
              <th>File</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="file in files" :key="file.id">
              <td>{{ file.fileName }}</td>
              <td>
                <a :href="file.filePath" target="_blank">{{ file.filePath }}</a>
              </td>
              <td>
                <button class="btn btn-delete" @click="deleteFile(file.id)">
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </section>
    </div>
  </div>
</template>

<script>


export default {
  name: "PatientDetails",
  data() {
    return {
      patient: null,
      files: [],
      apiFilePath: "",
      showForm: false,
      newFile: {
        fileName: "",
        filePath: "",
        description: "",
      },
      todos: [],
      showTodoForm: false,
      newTodo: {
        beschreibung: "",
        prioritaet: "",
        status: "",
      },
    };
  },
  methods: {
    toggleFileForm() {
      this.showForm = !this.showForm;
    },
    toggleTodoForm() {
      this.showTodoForm = !this.showTodoForm;
    },
    toggleTodoStatus(todo) {
      const newStatus = todo.status === "erledigt" ? "offen" : "erledigt";
      fetch(`https://webtechprojekt.onrender.com/todo/${todo.toDoId}/status/${newStatus}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        redirect: "follow",
      })
        .then(() => {
          this.loadPatientTodos(); // Lädt die Todo-Liste erneut, um die Änderungen anzuzeigen
        })
        .catch((error) => {
          console.error("Error updating todo status:", error);
        });
    },
    async loadPatientDetails() {
      try {
        const patientId = this.$route.params.id;
        const response = await this.getPatient(patientId);
        console.log(response);
        this.patient = response;
      } catch (error) {
        console.error("Error fetching patient details:", error);
      }
    },
    async getPatient(patientId) {
    try {
      const response = await fetch(`https://webtechprojekt.onrender.com/patient/${patientId}`);
      if (!response.ok) {
        throw new Error('Netzwerk-Antwort war nicht ok');
      }
      return await response.json();
    } catch (error) {
      console.error('Fehler beim Abrufen der Patientendetails:', error);
    }
  },
    async loadPatientFiles() {
  try {
    const patientId = this.$route.params.id;
    const response = await fetch(`https://webtechprojekt.onrender.com/patient/${patientId}/files`);
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    this.files = await response.json();
  } catch (error) {
    console.error("Error fetching patient files:", error);
  }
},
async deleteFile(fileId) {
  try {
    const patientId = this.$route.params.id;
    const response = await fetch(`https://webtechprojekt.onrender.com/file/${fileId}/patient/${patientId}`, {
      method: 'DELETE'
    });

    if (!response.ok) {
      throw new Error('Network response was not ok');
    }

    this.loadPatientFiles();
  } catch (error) {
    console.error("Error deleting file:", error);
  }
},

    addFile() {
      setTimeout(() => {
        console.log(this.apiFilePath);
        console.log(this.newFile.filePath);
        this.newFile.filePath = this.apiFilePath;
        console.log(this.newFile.filePath);
        const requestOptions = {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.newFile),
          redirect: "follow",
        };

        fetch("https://webtechprojekt.onrender.com/file", requestOptions)
          .then((response) => response.json())
          .then((result) => {
            console.log(result);
            // Aufruf der assignFileToPatient Methode mit der ID der gerade hinzugefügten Datei
            this.assignFileToPatient(result.id).then(() => {
              this.loadPatientFiles();
              this.showForm = false;
              this.resetNewFile();
            });
          })
          .catch((error) => {
            console.error("Error adding file:", error);
          });
      }, 60000);
    },
    handleFileUpload(event) {
  let uploadedFile = event.target.files[0];

  let reader = new FileReader();
  reader.onload = () => {
    let base64File = reader.result.split(",")[1];

    let data = {
      Parameters: [
        {
          Name: "File",
          FileValue: {
            Name: uploadedFile.name,
            Data: base64File,
          },
        },
        {
          Name: "StoreFile",
          Value: true,
        },
      ],
    };

    fetch("https://v2.convertapi.com/convert/pdf/to/html?Secret=grRqdvviof9pLfcD", {
      method: 'POST',
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data)
    })
    .then(response => response.json())
    .then(data => {
      console.log(data);
      this.apiFilePath = data.Files[0].Url; // Speichern Sie die URL als filePath
      console.log(this.apiFilePath);
    })
    .catch((error) => {
      console.log(error);
    });
  };
  reader.readAsDataURL(uploadedFile);
},

    assignFileToPatient(fileId) {
      if (!this.patient || !this.patient.id) {
        console.error("Patient ID is not available");
        return Promise.reject("Patient ID is not available");
      }

      const requestOptions = {
        method: "PUT", // PUT Methode zum Zuordnen der Datei
        body: "",
        redirect: "follow",
      };

      return fetch(
        `https://webtechprojekt.onrender.com/file/${fileId}/patient/${this.patient.id}`,
        requestOptions
      )
        .then((response) => response.text())
        .then((result) => console.log("File assigned to patient:", result))
        .catch((error) =>
          console.error("Error assigning file to patient:", error)
        );
    },

    resetNewFile() {
      this.newFile = {
        fileName: "",
        filePath: "",
        description: "",
      };
    },

    async loadPatientTodos() {
      try {
        const patientId = this.$route.params.id;
        const response = await fetch(
          `https://webtechprojekt.onrender.com/patient/${patientId}/todos`,
          {
            method: "GET", // Changed from PUT to GET to fetch data
            redirect: "follow",
          }
        );
        if (response.ok) {
          this.todos = await response.json(); // Update todos with the fetched data
        } else {
          throw new Error("Failed to fetch todos");
        }
      } catch (error) {
        console.error("Error fetching patient todos:", error);
      }
    },
    async deleteTodo(todoId) {
      try {
        const patientId = this.$route.params.id;
        await fetch(
          `https://webtechprojekt.onrender.com/todo/${todoId}/patient/${patientId}`,
          {
            method: "DELETE",
            headers: { "Content-Type": "application/json" },
            redirect: "follow",
          }
        );
        this.loadPatientTodos(); // Lädt die Todo-Liste erneut, um die Änderungen anzuzeigen
      } catch (error) {
        console.error("Error deleting todo:", error);
      }
    },
    async addTodo() {
      const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(this.newTodo),
        redirect: "follow",
      };

      fetch("https://webtechprojekt.onrender.com/todo", requestOptions)
        .then((response) => response.json())
        .then((result) => {
          console.log(result);
          //test
          // Aufruf der assignFileToPatient Methode mit der ID der gerade hinzugefügten Datei
          this.assignTodoToPatient(result.toDoId).then(() => {
            this.loadPatientTodos();
            this.showForm = false;
            this.resetNewTodo();
          });
        })
        .catch((error) => {
          console.error("Error adding file:", error);
        });
    },
    assignTodoToPatient(todoId) {
      if (!this.patient || !this.patient.id) {
        console.error("Patient ID is not available");
        return Promise.reject("Patient ID is not available");
      }

      const requestOptions = {
        method: "PUT", // PUT Methode zum Zuordnen der Datei
        body: "",
        redirect: "follow",
      };

      return fetch(
        `https://webtechprojekt.onrender.com/todo/${todoId}/patient/${this.patient.id}`,
        requestOptions
      )
        .then((response) => response.text())
        .then((result) => console.log("File assigned to patient:", result))
        .catch((error) =>
          console.error("Error assigning file to patient:", error)
        );
    },

    resetNewTodo() {
      this.newTodo = {
        beschreibung: "",
        prioritaet: "",
        status: "",
      };
    },
  },
  computed: {
    sortedTodos() {
      const priorityMap = { hoch: 1, mittel: 2, niedrig: 3 };

      return this.todos.slice().sort((a, b) => {
        if (a.status === "erledigt" && b.status === "erledigt") return 0;
        if (a.status === "erledigt") return 1;
        if (b.status === "erledigt") return -1;
        return priorityMap[a.prioritaet] - priorityMap[b.prioritaet];
      });
    },
  },
  created() {
    this.loadPatientDetails();
    this.loadPatientFiles();
    this.loadPatientTodos();
  },
};
</script>

<style scoped>
.container {
  padding: 20px;
}

.patient-header h2 {
  margin-bottom: 20px;
}

.info-box span {
  display: block;
  margin-bottom: 10px;
}

.file-table,
.todo-table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
}

.file-table th,
.file-table td,
.todo-table th,
.todo-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: left;
  max-width: none; /* Setzt eine maximale Breite für Zellen */
  overflow: hidden; /* Verhindert Überlaufen von Inhalten */
  text-overflow: ellipsis; /* Fügt '...' am Ende von überlangen Inhalten hinzu */
  white-space: nowrap;
}

.file-table th,
.todo-table th {
  background-color: #f2f2f2;
}

.btn, .btn-delete {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
}

.btn:hover {
  background-color: #0056b3;
}

.btn-delete {
  background-color: #dc3545;
}

.btn-delete:hover {
  background-color: #c82333;
}

.todo-done {
  text-decoration: line-through;
}

.todo-form,
.file-form {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 20px;
  background-color: #f2f2f2;
  margin-bottom: 20px;
}

.patient-details-head,
.extra-box {
  display: flex;
  justify-content: space-between;
  background-color: #f2f2f2;
  border: 1px solid #ccc;
  padding: 10px;
  margin: 5px;
}

/* Reaktionsfähige Anpassungen für mobile Geräte */
@media screen and (max-width: 600px) {
  .todo-form, .file-form, .patient-details-head, .extra-box {
    flex-direction: column;
    align-items: stretch;
  }

  .btn, .btn-delete {
    width: auto;
    height: auto;
    margin: 10px 0;
  }

  .file-table, .todo-table {
    display: block;
    overflow-x: auto;
  }
  .file-table td {
    white-space: normal; /* Erlaubt den Umbruch von Text in Zellen */
    word-wrap: break-word; /* Erzwingt den Umbruch von langen Wörtern/URLs */
    max-width: 150px; /* Entfernt die maximale Breite auf kleinen Bildschirmen */
    white-space: normal; /* Erlaubt den Umbruch von Text in Zellen */
    word-wrap: break-word;
  }
  .todo-table th {
    white-space: normal; /* Erlaubt den Umbruch von Text in Zellen */
    word-wrap: break-word; /* Erzwingt den Umbruch von langen Wörtern/URLs */
    max-width: none; /* Entfernt die maximale Breite auf kleinen Bildschirmen */
    white-space: normal; /* Erlaubt den Umbruch von Text in Zellen */
    word-wrap: break-word;
  }


}
</style>
