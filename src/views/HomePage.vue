<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
  <ion-title>Trello UTN</ion-title>
  <ion-buttons slot="end">
    <ion-button v-if="isAuthenticated" @click="logout">Cerrar sesión</ion-button>
  </ion-buttons>
</ion-toolbar>

    </ion-header>

    <ion-content>
      <!-- LOGIN -->
    <ion-card v-if="!isAuthenticated && !showRegister" class="auth-card">
      <ion-card-header>
        <ion-card-title class="ion-text-center">Iniciar Sesión</ion-card-title>
      </ion-card-header>
      <ion-card-content>
        <ion-item>
          <ion-icon name="mail-outline" slot="start"></ion-icon>
          <ion-input v-model="email" placeholder="Correo Electrónico" type="email"></ion-input>
        </ion-item>
        <ion-item>
          <ion-icon name="lock-closed-outline" slot="start"></ion-icon>
          <ion-input v-model="password" placeholder="Contraseña" type="password"></ion-input>
        </ion-item>
        <ion-button expand="block" @click="login" color="primary">Ingresar</ion-button>
        <ion-button expand="block" fill="clear" @click="showRegister = true">Crear Cuenta</ion-button>
      </ion-card-content>
    </ion-card>

    <!-- REGISTRO -->
    <ion-card v-if="!isAuthenticated && showRegister" class="auth-card">
      <ion-card-header>
        <ion-card-title class="ion-text-center">📝 Registro</ion-card-title>
      </ion-card-header>
      <ion-card-content>
        <ion-item>
          <ion-icon name="person-outline" slot="start"></ion-icon>
          <ion-input v-model="registerName" placeholder="Nombre completo"></ion-input>
        </ion-item>
        <ion-item>
          <ion-icon name="mail-outline" slot="start"></ion-icon>
          <ion-input v-model="registerEmail" placeholder="Correo Electrónico" type="email"></ion-input>
        </ion-item>
        <ion-item>
          <ion-icon name="lock-closed-outline" slot="start"></ion-icon>
          <ion-input v-model="registerPassword" placeholder="Contraseña" type="password"></ion-input>
        </ion-item>
        <ion-button expand="block" @click="register" color="primary">Registrar</ion-button>
        <ion-button expand="block" fill="clear" color="medium" @click="showRegister = false">Cancelar</ion-button>
      </ion-card-content>
    </ion-card>


      <!-- DASHBOARD -->
      <div v-if="isAuthenticated" class="ion-padding">
        <ion-segment v-model="activeTab" value="projects">
          <ion-segment-button value="projects">
            <ion-label>Proyectos</ion-label>
          </ion-segment-button>
          <ion-segment-button value="tasks" :disabled="!selectedProject">
            <ion-label>Tareas</ion-label>
          </ion-segment-button>
        </ion-segment>

        <!-- PROYECTOS -->
        <div v-if="activeTab === 'projects'" class="ion-margin-top">
          <ion-card>
            <ion-card-header>
              <ion-card-title>Mis Proyectos</ion-card-title>
            </ion-card-header>
            <ion-card-content>
              <ion-list>
                <ion-item v-for="project in projects" :key="project.id" button @click="selectProject(project.id)">
  <ion-label>
    <h2>{{ project.name }}</h2>
    <p>{{ project.description }}</p>
  </ion-label>
  <ion-buttons slot="end">
    <ion-button color="medium" @click.stop="openEditModal(project)">Editar</ion-button>
    <ion-button color="danger" @click.stop="deleteProject(project.id)">Eliminar</ion-button>
  </ion-buttons>
</ion-item>

              </ion-list>
              <ion-title size="small">Agregar Nuevo Proyecto</ion-title>
              <ion-item>
                <ion-input v-model="newProject" placeholder="Nombre del Proyecto" label="Nombre" label-placement="floating"></ion-input>
              </ion-item>
              <ion-item>
                <ion-input v-model="newProjectDescription" placeholder="Descripción del Proyecto" label="Descripción" label-placement="floating"></ion-input>
              </ion-item>
              <ion-button expand="full" @click="createProject">Agregar Proyecto</ion-button>
              <!-- Modal para editar proyecto -->
<ion-modal :is-open="showEditModal" @did-dismiss="closeEditModal">
  <ion-header>
    <ion-toolbar>
      <ion-title>Editar Proyecto</ion-title>
      <ion-buttons slot="end">
        <ion-button @click="closeEditModal">Cerrar</ion-button>
      </ion-buttons>
    </ion-toolbar>
  </ion-header>
  <ion-content class="ion-padding">
    <ion-item>
      <ion-input v-model="editProjectName" label="Nombre" label-placement="floating"></ion-input>
    </ion-item>
    <ion-item>
      <ion-input v-model="editProjectDescription" label="Descripción" label-placement="floating"></ion-input>
    </ion-item>
    <ion-button expand="full" @click="saveProjectChanges">Guardar Cambios</ion-button>
  </ion-content>
</ion-modal>

            </ion-card-content>
          </ion-card>
        </div>

       <!-- TAREAS -->
<div v-if="activeTab === 'tasks' && selectedProject" class="ion-margin-top">
  <ion-card>
    <ion-card-header>
      <ion-card-title>Tareas del Proyecto</ion-card-title>
      <ion-note class="ion-text-end">Proyecto #{{ selectedProject }}</ion-note>
    </ion-card-header>
    <ion-card-content>
      <ion-list>
        <ion-item v-for="task in tasks" :key="task.id">
          <ion-label>
            <h2 @click="editField('name', task)">{{ task.name }}</h2>
            <p @click="editField('description', task)">{{ task.description }}</p>
            <div>
  <strong>Asignados a:</strong>
  <ul>
    <li v-for="user in task.users" :key="user.id">{{ user.name }}</li>
  </ul>
</div>

            <strong>Estado: </strong>
            <template v-if="editingTaskStatusId === task.id">
              <ion-select v-model="task.status" @ionChange="updateTaskStatus(task)">
                <ion-select-option value="Pendiente">Pendiente</ion-select-option>
                <ion-select-option value="En progreso">En progreso</ion-select-option>
                <ion-select-option value="Completada">Completada</ion-select-option>
              </ion-select>
            </template>
            <span v-else @click="editField('status', task)">{{ task.status }}</span>
            <div class="ion-padding-top">
              <ion-item>
                <ion-label>Asignar a</ion-label>
                <ion-select v-model="task.assigned_user_id" @ionChange="assignUserToTask(task)">
                  <ion-select-option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</ion-select-option>
                </ion-select>
              </ion-item>
            </div>
          </ion-label>
          <ion-button @click="deleteTask(task.id)" slot="end" color="danger">Eliminar</ion-button>
        </ion-item>
      </ion-list>

      <ion-title size="small">Agregar Nueva Tarea</ion-title>
      <ion-item>
        <ion-input v-model="newTask" placeholder="Nombre de la Tarea" label="Nombre" label-placement="floating"></ion-input>
      </ion-item>
      <ion-item>
        <ion-input v-model="newTaskDescription" placeholder="Descripción de la Tarea" label="Descripción" label-placement="floating"></ion-input>
      </ion-item>
      <ion-item>
        <ion-select v-model="newTaskStatus" placeholder="Estado" label="Estado" label-placement="floating">
          <ion-select-option value="Pendiente">Pendiente</ion-select-option>
          <ion-select-option value="En progreso">En progreso</ion-select-option>
          <ion-select-option value="Completada">Completada</ion-select-option>
        </ion-select>
      </ion-item>
      <ion-item>
  <ion-label>Asignar a</ion-label>
  <ion-select v-model="task.assigned_user_ids" multiple @ionChange="assignUsersToTask(task)">
    <ion-select-option v-for="user in users" :key="user.id" :value="user.id">
      {{ user.name }}
    </ion-select-option>
  </ion-select>
</ion-item>


      <ion-button expand="full" @click="createTask">Agregar Tarea</ion-button>
    </ion-card-content>
  </ion-card>
</div>

      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import axios from 'axios';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonSegment, IonSegmentButton, IonLabel, IonList, IonItem,
  IonButton, IonInput, IonCard, IonCardHeader, IonCardTitle,
  IonCardContent, IonNote, IonSelect, IonSelectOption,
} from '@ionic/vue';

export default {
  components: {
    IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
    IonSegment, IonSegmentButton, IonLabel, IonList, IonItem,
    IonButton, IonInput, IonCard, IonCardHeader, IonCardTitle,
    IonCardContent, IonNote, IonSelect, IonSelectOption,
  },
  data() {
    return {
      email: '',
      password: '',
      token: null,
      userId: null,
      isAuthenticated: false,
      projects: [],
      tasks: [],
      newProject: '',
      newProjectDescription: '',
      newTask: '',
      newTaskDescription: '',
      newTaskStatus: 'Pendiente',
      selectedProject: null,
      activeTab: 'projects',
      editingTaskStatusId: null,
      showEditModal: false,
editProjectId: null,
editProjectName: '',
editProjectDescription: '',
registerName: '',
registerEmail: '',
registerPassword: '',
showRegister: false,


    };
  },
  methods: {
    async login() {
      try {
        const res = await axios.post('https://proyecto1-production-a06f.up.railway.app/api/login', {
          email: this.email,
          password: this.password,
        });
        this.token = res.data.token;
        this.userId = res.data.user.id;
        this.isAuthenticated = true;
    await this.fetchUsers();
    await this.fetchProjects();
      } catch (err) {
        alert('Error al iniciar sesión');
      }
    },
    getUserName(id) {
    const user = this.users.find(u => u.id === id);
    return user ? user.name : 'Desconocido';
  },



  async assignUsersToTask(task) {
  try {
    await fetch(`https://proyecto1-production-a06f.up.railway.app/api/tasks/${task.id}/assign-users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      },
      body: JSON.stringify({
        user_ids: task.assigned_user_ids
      })
    });
  } catch (error) {
    console.error('Error al asignar usuarios:', error);
  }
}


,
    async register() {
  try {
    const res = await axios.post('https://proyecto1-production-a06f.up.railway.app/api/register', {
      name: this.registerName,
      email: this.registerEmail,
      password: this.registerPassword,
    });

    alert('Usuario registrado correctamente. Ahora puedes iniciar sesión.');
    this.registerName = '';
    this.registerEmail = '';
    this.registerPassword = '';
    this.showRegister = false;
  } catch (err) {
    console.error(err);
    alert('Error al registrar usuario');
  }
},
async logout() {
    this.token = null;
    this.isAuthenticated = false;
    this.selectedProject = null;
    this.projects = [];
    this.tasks = [];
    this.$router.push('/login');
  },
    async fetchProjects() {
      try {
        const res = await axios.get('https://proyecto1-production-a06f.up.railway.app/api/projects', {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.projects = res.data;
      } catch (err) {
        console.error(err);
      }
    },
    async createProject() {
      if (!this.newProject) return;
      try {
        await axios.post('https://proyecto1-production-a06f.up.railway.app/api/projects', {
          name: this.newProject,
          description: this.newProjectDescription,
          user_id: this.userId,
        }, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.newProject = '';
        this.newProjectDescription = '';
        this.fetchProjects();
      } catch (err) {
        console.error(err);
      }
    },
    async deleteProject(id) {
      try {
        await axios.delete(`https://proyecto1-production-a06f.up.railway.app/api/projects/${id}`, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.fetchProjects();
        if (this.selectedProject === id) {
          this.selectedProject = null;
          this.tasks = [];
        }
      } catch (err) {
        console.error(err);
      }
    },
    async selectProject(projectId) {
      this.selectedProject = projectId;
      this.activeTab = 'tasks';
      this.fetchTasks(projectId);
    },

    openEditModal(project) {
  this.editProjectId = project.id;
  this.editProjectName = project.name;
  this.editProjectDescription = project.description;
  this.showEditModal = true;
},
closeEditModal() {
  this.showEditModal = false;
},
async saveProjectChanges() {
  try {
    await axios.put(`https://proyecto1-production-a06f.up.railway.app/api/projects/${this.editProjectId}`, {
      name: this.editProjectName,
      description: this.editProjectDescription,
    }, {
      headers: { Authorization: `Bearer ${this.token}` },
    });
    this.closeEditModal();
    this.fetchProjects();
  } catch (err) {
    console.error(err);
    alert("Error al actualizar el proyecto");
  }
},

    async fetchTasks(projectId) {
      try {
        const res = await axios.get(`https://proyecto1-production-a06f.up.railway.app/api/projects/${projectId}/tasks`, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.tasks = res.data;
      } catch (err) {
        console.error(err);
      }
    },
    async createTask() {
      if (!this.newTask || !this.selectedProject) return;
      try {
        await axios.post(`https://proyecto1-production-a06f.up.railway.app/api/projects/${this.selectedProject}/tasks`, {
          name: this.newTask,
          description: this.newTaskDescription,
          status: this.newTaskStatus,
          user_ids: [this.userId],
        }, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.newTask = '';
        this.newTaskDescription = '';
        this.newTaskStatus = 'Pendiente';
        this.fetchTasks(this.selectedProject);
      } catch (err) {
        console.error(err);
        alert('Error al crear la tarea');
      }
    },
    async deleteTask(id) {
      try {
        await axios.delete(`https://proyecto1-production-a06f.up.railway.app/api/tasks/${id}`, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.fetchTasks(this.selectedProject);
      } catch (err) {
        console.error(err);
      }
    },
    editField(field, task) {
      if (field === 'status') {
        this.editingTaskStatusId = task.id;
      } else {
        const newValue = prompt(`Editar ${field}`, task[field]);
        if (newValue !== null) {
          this.updateTaskField(task.id, field, newValue);
        }
      }
    },
    async updateTaskField(taskId, field, value) {
      try {
        await axios.put(`https://proyecto1-production-a06f.up.railway.app/api/tasks/${taskId}`, {
          [field]: value,
        }, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
        this.fetchTasks(this.selectedProject);
      } catch (err) {
        console.error(err);
      }
    },
    async updateTaskStatus(task) {
      try {
        await this.updateTaskField(task.id, 'status', task.status);
        this.editingTaskStatusId = null;
      } catch (err) {
        console.error(err);
      }
    },
    async fetchUsers() {
    try {
      const response = await fetch('https://proyecto1-production-a06f.up.railway.app/api/users', {
        headers: {
          'Authorization': `Bearer ${this.token}` 
        }
      });
      const data = await response.json();
      this.users = data;
    } catch (error) {
      console.error('Error al obtener los usuarios:', error);
    }
  },

    
  },
};
</script>
<style scoped>
.auth-card {
  max-width: 400px;
  margin: 40px auto;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 4px 10px rgba(0,0,0,0.1);
}

ion-card-title {
  font-size: 1.3rem;
}

ion-item {
  margin-bottom: 10px;
}

/* Proyecto y tarea */
ion-card {
  max-width: 600px;
  margin: 20px auto;
  border-radius: 12px;
  box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.1);
}

ion-card-header {
  background-color: #f4f4f9;
  border-radius: 10px 10px 0 0;
  padding: 15px;
}

ion-card-content {
  padding: 15px;
}

ion-title {
  font-size: 1.1rem;
  color: #333;
}

ion-button {
  margin-top: 10px;
}

ion-label {
  font-size: 1rem;
  color: #555;
}

ion-input,
ion-select {
  margin-bottom: 15px;
}

ion-segment {
  margin-top: 20px;
  text-align: center;
}

ion-segment-button {
  font-size: 1rem;
  padding: 10px 20px;
}

ion-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
}

ion-button[color="danger"] {
  background-color: #e74c3c;
  color: white;
}

ion-button[color="medium"] {
  background-color: #95a5a6;
  color: white;
}

ion-card-content ion-list {
  padding: 0;
}
</style>


